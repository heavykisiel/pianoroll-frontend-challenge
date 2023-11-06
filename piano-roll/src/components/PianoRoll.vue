<template>
  <div>
    <h1 v-if="PianoRollsArray.length === 0">Click here to load Piano Rolls!</h1>
    <h1 v-else-if="activeList.length === 0">Select your Piano Roll</h1>
    <h1 v-else>Explore your Piano Roll</h1>
    <div id="button-container">
      <button v-if="PianoRollsArray.length === 0" id="loadCSV" @click="handleClickforDataLoader">Load Piano Rolls!</button>
      <button v-if="activeIndex !== null" @click="enterGrid()">Enter grid view</button>
      <button v-if="activeIndex !== null" @click="toggleSelection()">Clear Selection</button>
    </div>
    <div class="piano-roll-container" :class="{ pianoRollSelected: activeList.length > 0}">
      <div v-if="activeList.length == 0" class="gridContainerWrapper">
        <div v-for="(item, index) in PianoRollsArray" :key="index" class="piano-roll-card"
            @click="toggleActive(item, index)">
          <div v-html="item.svgElement.outerHTML"></div>
        </div>
      </div>
        <div v-if="activeIndex !== null" class="piano-roll-card  active">
          <div class="piano-roll-svg active">
          <div class="active-element"
            @mousedown="startSelection(activeIndex, $event)"
            @mousemove="extendSelection(activeIndex, $event)"
            @mouseup="endSelection(activeIndex, $event)" 
            @mouseleave="endSelection(activeIndex, $event)">
            <div v-html="PianoRollsArray[activeIndex].svgElement.outerHTML"></div>     
            <div class="selection-line" v-if="isSelecting " :style="selectionLineStyle"></div>
            <div class="painted-area" v-if="isSelecting "  :style="paintedAreaStyle"></div>
            <div class="selected-lines" v-if="(startNoteIndex !== null || endNoteIndex !== null) && lineSwitch"
            :style="selectedLineLeftStyle"></div>
            <div class="selected-lines" v-if="(startNoteIndex !== null || endNoteIndex !== null) && lineSwitch"
            :style="selectedLineRightStyle"></div>
          </div>
        </div>
        <div class="description"></div>  
        <div v-if="null !== activeIndex">there are {{notes.length}} notes selected</div>
        <div v-if="null !== activeIndex && notes.length > 0">first point is selected {{startNoteIndex}} px</div>
        <div v-if="null !== activeIndex && notes.length > 0">second point is selected {{Math.round(endNoteIndex)}} px</div>
        
        </div>
        <div v-if="activeIndex !== null">
        <div class="list-inactives">
          <div v-for="(item ,index) in inactiveList" :key="index.id" class="piano-roll-card inactive"
              @click="toggleActive(item, index)"
              >                
              <div v-html="item.svgElement.outerHTML"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';
import PianoRoll from '@/utils/PianoRollLogic';

export default {
  setup() {
    const pianoRollCount = 32;
    const activeIndex = ref(null);
    const PianoRollsArray = ref([]);
    const csvURL = 'https://pianoroll.ai/random_notes';
    const data = ref(null);
    const notes = ref([]);

    const isSelecting = ref(false); 
    const startNoteIndex = ref(null); 
    const endNoteIndex = ref(null); 
    const lineSwitch = ref(false);
    
    const toggleActive = (item, index) => {
      if (activeIndex.value === null || activeIndex.value !== index) {
        activeIndex.value = index;
      } 
      else {
        activeIndex.value = index +1 ;
      }
      toggleSelection();
    };

    const toggleSelection = () => {
        lineSwitch.value =!lineSwitch.value;
        startNoteIndex.value = null;
        endNoteIndex.value = null;
        notes.value = [];
    }
    const enterGrid = () => {
        activeIndex.value = null;
    };

    const loadPianoRollData = async () => {
      try {
        const response = await fetch(csvURL);
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
        data.value = await response.json();
      } catch (error) {
        console.error('Error loading data:', error);
      }
    };
    const loadData =()=> {
      const buff = [];
      
      for (let it = 0; it < pianoRollCount; it++) {
        const start = it * 60;
        const end = start + 60;
        const partData = data.value.slice(start, end);

        const { svg } = preparePianoRollCard(it);
        const roll = new PianoRoll(svg, partData);
        buff.push(roll);
      }
      PianoRollsArray.value = buff;
    }

    const handleClickforDataLoader = async () => {
      if (!data.value) await loadPianoRollData();
      if (!data.value) {
        throw new Error('Error loading data!');
      }
      loadData();
    };

    const preparePianoRollCard = (rollId) => {
      const cardDiv = document.createElement('div');
      cardDiv.classList.add('piano-roll-card');

      const descriptionDiv = document.createElement('div');
      descriptionDiv.classList.add('description');
      descriptionDiv.textContent = `This is a piano roll number ${rollId}`;

      const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
      svg.classList.add('piano-roll-svg' + rollId);
      svg.style.width='100%';

      cardDiv.appendChild(svg);
      cardDiv.appendChild(descriptionDiv);

      return { cardDiv, svg };
    };

    const startSelection = (index, event) => {
      if (activeIndex.value === index ) {
            isSelecting.value = true;
            startNoteIndex.value = event.layerX;
      }
    };

    const extendSelection = (index, event) => {
      if(isSelecting.value && activeIndex.value === index) {
        if(event.target.matches('.piano-roll-svg' + activeIndex.value)) {
          endNoteIndex.value = event.layerX;
          }
        else if(event.target.matches('.painted-area')) {
          endNoteIndex.value = event.clientX - event.currentTarget.getBoundingClientRect().left;
          }
        else if(event.target.matches('rect')) {
          endNoteIndex.value = event.clientX - event.currentTarget.getBoundingClientRect().left;
          }
        else if(event.target.matches('line')) {
          endNoteIndex.value = event.clientX - event.currentTarget.getBoundingClientRect().left;
      }
    }
    };

    const endSelection = (index) => {
      if (activeIndex.value === index && isSelecting.value) {
        isSelecting.value = false;

        const containerDiv = document.querySelector('.piano-roll-svg' + activeIndex.value);
        const noteElements = containerDiv.querySelectorAll('.note-rectangle');

        let SelectedNotes = [];
        noteElements.forEach((element) => {
            const elementRect = element.getBoundingClientRect();
            const elementX = Math.round(elementRect.left);
            if ((startNoteIndex.value <= elementX && endNoteIndex.value >= elementX) ||
                (startNoteIndex.value >= elementX && endNoteIndex.value <= elementX)) {
                SelectedNotes.push(element);
            }
        });
        lineSwitch.value = true;
        return notes.value = SelectedNotes;
      }
    };

    const selectionLineStyle = computed(() => {
        if (isSelecting.value) {
        const left = Math.min(startNoteIndex.value, endNoteIndex.value) + 'px';
          return { left };
        } else {
          return { display: 'none' };
        }
    });

    const paintedAreaStyle = computed(() => {
        if (isSelecting.value) {
            const left = Math.min(startNoteIndex.value, endNoteIndex.value) + 'px';
            const width = Math.abs(startNoteIndex.value - endNoteIndex.value) + 'px';
            return { left, width };
        } else {
            return { display: 'none' };
        }
    });

    const selectedLineLeftStyle = computed(() => {
      const left = Math.min(startNoteIndex.value, endNoteIndex.value) + 'px';
      return { left }
    });

    const selectedLineRightStyle = computed(() => {
      const left = Math.max(startNoteIndex.value, endNoteIndex.value) + 'px';
      return { left }
    });

    const activeList = computed(() => {
      if (activeIndex.value !== null) {
        return [PianoRollsArray.value[activeIndex]];
      }
      return [];
    });

    const inactiveList = computed(() => {
      if (activeIndex.value !== null) {
        return PianoRollsArray.value.filter((_, index) => index !== activeIndex.value);
      }
      return PianoRollsArray.value;
    });

    return {
      handleClickforDataLoader,
      pianoRollCount,
      toggleActive,
      PianoRollsArray,
      activeIndex,
      startSelection,
      endSelection,
      extendSelection,
      notes,
      paintedAreaStyle,
      selectionLineStyle,
      selectedLineLeftStyle,
      selectedLineRightStyle,
      isSelecting,
      startNoteIndex,
      endNoteIndex,
      inactiveList,
      activeList,
      enterGrid,
      lineSwitch,
      toggleSelection
    };
  },
};
</script>
<style>
.piano-roll-container .gridContainerWrapper {
  display: grid;
  gap: 3.5vw;
  margin: 1.5vh 5vw 0px 5vw;
  grid-template-columns: repeat(1, 1fr); 

  @media (min-width: 600px) {
    grid-template-columns: repeat(2, 1fr);
  }

  @media (min-width: 992px) {
    grid-template-columns: repeat(3, 1fr);
  }

  @media (min-width: 1200px) {
    grid-template-columns: repeat(4, 1fr); 
  }
}
.piano-roll-container.pianoRollSelected {
  display: flex;
  gap: 3.5vw;
  margin: 1.5vh 5vw 0px 5vw;
  flex-direction: column;

  @media (min-width: 600px) {
    flex-direction: column;
  }
  @media (min-width: 1080px) {
    flex-direction: row;
  }
}
h1 {
    margin-bottom: 20px;
    font-size: 42px;
}

button {
  padding: 15px 25px;
  font-size: 18px;
  color: #F0F0F0;
  background-color: #944038;
  border: none;
  cursor: pointer;
  transition: background-color 0.3s;
  border-radius: 5px;
  border-bottom: 3px solid #381815;  
  position: relative;  
  transition: all 1.1s ease;  
}
svg {
  aspect-ratio: 4/3;
  width: 100%;
  @media (min-width: 768px) {
    aspect-ratio: 4/3;        
  }
  @media (min-width: 800px) {
    aspect-ratio: 16/9;
  }
  @media (min-width: 1800px) {
    aspect-ratio: 21/9;
  }
}
#button-container {
  display: flex;
  justify-content: center;
  gap: 1.5vh;
}

button:hover {
  transform: scale(1.05);
}

.piano-roll-svg {
  border: 2px solid #381815;
}

.piano-roll-card {
  border: 1px solid #ccc;
  margin-bottom: 8px;
  padding: 10px;
  width: 100%;
  box-sizing: border-box;
  border-radius: 10px; /* Rounded corners */
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1); /* Shadow effect */
  background-color: #d0d9dc;
}
@keyframes slide-fade-in {
  from{
      opacity: 0;
      transform: translateY(5vh);
  }
}
@keyframes slide-fade-on {
  from{
      opacity: 0;
      transform: translateY(-5vh);
  }
}
@media (prefers-reduced-motion: no-preference) {

  .piano-roll-card,h1{
    view-timeline-name: --item-timeline;
    animation: slide-fade-in 1s both;
    animation-timeline:  --item-timeline;
    animation-range: contain 0% contain 50%;
  }
}
@media (prefers-reduced-motion: no-preference) {
h1{
  view-timeline-name: --item-timeline;
  animation: slide-fade-on 1s both;
  animation-timeline:  --item-timeline;
  animation-range: contain 0% contain 50%;
}
}
div.paino-roll-card:hover {
  transform: rotatex(15deg);
  transform: rotatez(15deg);
  transition:.6s;

}
.piano-roll-card.active {
  position: relative;
  height: fit-content;
}
.description {
  margin-top: 10px;
}
.active-element {
  position: relative;
}
.list-inactives{
  overflow-y:auto; 
  padding: 20px; 
  height: 80vh;
  scroll-behavior: smooth;
}
.selection-line {
    position: absolute;
    background-color: #00f; 
    width: 2px; 
    top: 0;
    bottom: 0;
  }

.painted-area {
    position: absolute;
    background-color: rgba(0, 0, 255, 0.2); 
    top: 0;
    bottom: 0;
  }

.selected-lines {
  position: absolute;
  background-color: rgb(0, 4, 255); 
  width: 2px; 
  top: 0;
  bottom: 0;

}
.loading-indicator {
  text-align: center;
  padding: 10px;
  background-color: #f0f0f0;
}


</style>
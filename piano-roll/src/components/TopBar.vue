<template>
  <nav class="navbar">
    <div class="logo-container">
      <img src="@/assets/white.svg" alt="Logo">
    </div>
  </nav>
  </template>
  
  <script>
  export default {
    name: 'TopBar'
  };
  </script>
  
  <style scoped>

  .navbar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    background-color: var(--main-primary-color);
    box-shadow: 0 2px 5px rgba(0,0,0,0.2);  /* optional drop shadow */
  }
  
  .logo-container img {
    height: 5vh;
    width: auto;
    fill: white;
  }
  


  </style>
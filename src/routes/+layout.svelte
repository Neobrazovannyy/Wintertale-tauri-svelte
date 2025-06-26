<script lang="ts">
    let { children } = $props();
</script>

<!-- Window Tabs -->
<div class="window_tabs">

  <div class="left_window_tabs">
    left Window Tabs
  </div>

  <div class="right_window_tabs">
    right Window Tabs
  </div>

</div>

<!-- Main Window & Size Menu -->
<main>
  <!-- Size Menu -->
  <div class="size_menu">

    <div class="side_tabs">
      side tabs
    </div>
    
    <div class="collapsible_tab">
      collapsible tab
    </div>

  </div>

  <!-- Main Window -->
  <div class="main_window">
    <!-- Wintertale -->
    {@render children()}
  </div>
</main>

<!-- Footer -->
<div class="footer">
  Footer
</div>

<style lang="scss">
  // font
  $absolute_font_family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  $absolute_font_weight: 300;
  // color
  $lite_blue_snow: rgb(63, 85, 118);  
  $dark_snow: rgb(40, 54, 75);
  $lite_color: rgb(200, 229, 255);
  $text_white: rgb(229, 242, 255);  // text
  // other const
  $width_size: 250px;


 *{
    background-color: $dark_snow;
    color: $text_white;
    font-size: 16px;
    font-family: $absolute_font_family;
    font-weight: $absolute_font_weight;
    border: 2px solid $lite_color;  /*del*/
  }
  

  /*=====================*/
  /*==== Window Tabs ====*/
  /*=====================*/
  .window_tabs{
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    height: 30px;
    font-size: 16px;
  }

  
  /*=====================*/
  /*==== Main Window ====*/
  /*=====================*/
  main{
    display: flex;
    flex-direction: row;
    font-size: 50px;
    flex: 1;

    .size_menu{
      display: flex;
      flex-direction: row;
      width: $width_size;

      .side_tabs{
        width: 40px;
      }
      .collapsible_tab{
        flex: 1;
      }
    }

    .main_window{
      flex: 1;
    }
  }
  
  /*================*/
  /*==== Footer ====*/
  /*================*/
  .footer{ 
    height: 20px;
    font-size: 16px;
  }
</style>
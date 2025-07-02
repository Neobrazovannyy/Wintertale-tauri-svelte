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

  <input type="text" class="mini_console" placeholder="CTRL+SHIFT">
</main>

<!-- Footer -->
<div class="footer">
  Footer
</div>

<style lang="scss">
  /*==font==*/
  $absolute_font_family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  $absolute_font_weight: 300;
  $letter_spacing: 1.6px;
  /*==color==*/
    // text & border
  $text_white: rgb(229, 240, 255);
  $text_and_border_lite_blue: #5b7bac;
    // border
  // $border_lite_blue: #5C7198;
    // background
  $bg_dark_grey: #161b24; // first choice
  $bg_dark_blue: #172030; // second choice
  /*==other const==*/
  $width_size: 250px;

  html, body {
    height: 100%;
    margin: 0;
  }

  *{
    background-color: $bg_dark_grey;
    color: $text_white;
    font-size: 16px;
    font-family: $absolute_font_family;
    font-weight: $absolute_font_weight;
    letter-spacing: $letter_spacing;
    border: 2px solid $text_and_border_lite_blue;  /*del*/
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
    flex: 1;
    display: flex;
    flex-direction: row;
    font-size: 50px;
    min-height: 0; 

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

    .mini_console{
      position: absolute;
      right: 0;
      bottom: 25px;
      height: 20px;

      &:focus{
        outline: none;
        box-shadow: none;
      }
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
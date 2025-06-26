<div class="falling_snow"></div>

<div class="welcome_window">
    <span class="title">
        welcom
    </span>
    <span class="to">
        to
    </span>
    <span class="name">
        Winter Tale
    </span>
</div>

<div class="button">
    <div class="box_btn box_create">
        <button class="btn_create">
            create
        </button>
        <span class="text">
            Create a new note
        </span>
    </div>
    <div class="box_btn box_open">
        <button class="btn_open">
            open
        </button>
        <span class="text">
            Open an existing note
        </span>
    </div>
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


    .falling_snow{
        position: absolute;
    }

    .welcome_window{
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 40%;
        font-size: $absolute_font_family;
        font-weight: $absolute_font_weight;

        .title{
            font-size: 50px;
            text-transform: uppercase;
        }
        .to{
            font-size: 30px;
        }
        .name{
            font-size: 50px;
        }
    }

    .button{
        display: flex;
        flex-direction: column;
        align-items: center;
        font-size: 25px;
        text-transform: uppercase;

        button{
            all: unset;
            cursor: pointer;
            background-color: $lite_color;
            color: $text_white;
            text-align: center;
            margin-bottom: 10px;
            border-radius: 5px;
        }


        .box_btn{
            display: flex;
            align-items: center;
            justify-content: space-between;
            border-bottom: 2px solid $lite_color;
            margin-bottom: 30px;
            width: 500px;
        }

        .box_create{
            .btn_create{
                width: 120px;
                height: 45px;
            }

            .text{
                font-size: 20px;
            }
        }
        
        .box_open{
            flex-direction: row-reverse;

            .btn_open{
                width: 120px;
                height: 45px;
            }
            .text{
                font-size: 20px;
            }
        }
    }

</style>
<script lang="ts">
    import { onMount } from "svelte";

    function getRandomValue(min: number, max: number, if_floor: boolean) : number{
        let random_value : number;

        random_value = Math.random() * (max - min + 1) + min;
        if(if_floor){
            random_value = Math.floor(random_value);
        }

        return random_value;
    }

    
onMount(() => {
    let delay_fall_snowflake: number = 0;
    let duration_fall_snowflake: number = 0;

    let CreateSnows = ()=>{

        setInterval(()=>{
            /*========================*/
            /*=== Create Snowflake ===*/
            /*========================*/
            delay_fall_snowflake = getRandomValue(1, 3, false) * 1000;
            duration_fall_snowflake = getRandomValue(6, 8, false) * 1000;
                
            let el_main_window = document.querySelector(".main_window");
            let el_falling_snow = document.querySelector(".falling_snow");
            let el_snowflake  = document.createElement("div");
            el_snowflake .className = "snowflake";

            el_main_window.style.position = "relative"; //ignore
            el_snowflake .style.position = "absolute";
            el_snowflake .style.borderRadius = "50%";
            el_snowflake .style.opacity = "0";
            el_snowflake .style.border = `${getRandomValue(2, 5, true)}px solid rgb(229, 242, 255)`;
            el_snowflake .style.right = `${getRandomValue(0, 100, false)}%`;

            el_snowflake .animate(
            [
                { top: "0%", opacity: 0, offset: 0 },
                { opacity: 1, offset: 0.6 },
                { top: "100%", opacity: 0, offset: 1 }
            ], {
                duration: duration_fall_snowflake,
                easing: "linear",
                delay: delay_fall_snowflake,
                fill: "forwards",
            });
    

            el_falling_snow?.appendChild(el_snowflake );
                
            /*========================*/
            /*=== Delate Snowflake ===*/
            /*========================*/
            setTimeout(()=>{
                
            }, duration_fall_snowflake+delay_fall_snowflake);

        }, delay_fall_snowflake);

    }
    
    CreateSnows();
});
</script>


<div class="falling_snow"></div>

<div class="welcome_window">
    <span class="title">
        welcome
    </span>
    <span class="to">
        to
    </span>
    <span class="name">
        Winter Tale
    </span>
</div>

<div class="links_window">
    <div class="links">
    
        <div class="box_links create_vault">
            <div class="title">
                Create Vault 
            </div>
            <div class="description">
                create a new local storage
            </div>
        </div>
    
        <div class="box_links create_note">
            <div class="title">
                Open Vault
            </div>
            <div class="description">
                open another repository
            </div>
        </div>
    
        <div class="box_links create_folder">
            <div class="title">
                Create Folder
            </div>
            <div class="description">
                create a folder to structure your notes
            </div>
        </div>
    
        <div class="box_links settings">
            <div class="title">
                Settings
            </div>
            <div class="description">
                simple settings for the app
            </div>
        </div>
    
        <div class="box_links tutorial">
            <div class="title">
                Tutorial
            </div>
            <div class="description">
               quick and easy tutorial 
            </div>
        </div>
    </div>
</div>

<!--
Create Vault. create a new local storage
Создать хранилище. создать новое локальное хранилище 

Open repository. open another repository
Открыть хранилище. открыть другое хранилище 

Create Folder. Create a folder to structure your notes
Создать Папку. Создать папку для структуризации заметок

Settings. simple settings for the app
Настройки. простые настройки для приложения 

Tutorial. Quick and easy tutorial
Обучение. Быстрое и простое обучение
-->

<style lang="scss">
    /* font */
    // $absolute_font_family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    // $absolute_font_weight: 300;
    /* color */
    $lite_blue_snow: rgb(63, 85, 118);
    $text_white: rgb(229, 242, 255);  // text


    *{  //del
        // border: 2px solid $text_white;
    }

    .welcome_window{
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: end;
        height: 40%;
        letter-spacing: 5px;

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

    .links_window{
        display: flex;
        flex-direction: column;
        align-items: center;
        width: 100%;
        height: 60%;

        .links{
            display: flex;
            flex-direction: column;
            width: 600px;
            margin-top: 50px;
    
            .box_links{
                display: flex;
                flex-direction: row;
                justify-content: space-between;
                align-items: flex-end;
                padding-bottom: 5px;
                margin-bottom: 20px;
                border-bottom: 2px solid $text_white;
                
                .title{
                    font-size: 25px;
                    color: $lite_blue_snow;
                    letter-spacing: 2.5px;
                    text-transform: uppercase;
                }
                .description{
                    font-size: 16px;
                    letter-spacing: 1.6px;

                }
                
            }
        }

    }

</style>
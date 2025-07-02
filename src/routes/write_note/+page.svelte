<script lang="ts">
    // let m_textarea_text: string = $state("");
    // let m_count_input: number = $state(0);
    let m_textarea_id: string[] = $state(["textarea_id_1"]);

    function L(description: string = "", variable: string): void {
        console.log(`${description}: ${variable}`);
    }

    async function HandleInputBlockWrite(id_name_element: string): Promise<void> {
        /*==== Type Element ====*/
        await CheckTypeElement(id_name_element);
    }

    /*======================*/
    /*==== Type Element ====*/
    /* 1. Symbol            */
    /* 2. Line              */
    /* 3. Block             */
    /*======================*/
    async function CheckTypeElement(id_name_element: string): Promise<void> {

        let el_textarea = document.querySelector(`#${id_name_element}`) as HTMLTextAreaElement | null;
        let cursor_pos_start: number = el_textarea?.selectionStart || 0;


        let Symbol = async (el_textarea: HTMLTextAreaElement | null, cursor_pos_start: number): Promise<void> =>{
            await CheckTypeList(el_textarea, cursor_pos_start);
        }

        let Line = async (id_name_element: string): Promise<void> =>{
            
        }

        let Block = async (id_name_element: string): Promise<void> =>{
            
        }

        await Symbol(el_textarea, cursor_pos_start);
        await Line(id_name_element);    //! el_textarea
        await Block(id_name_element);   //! el_textarea
    }

    async function CheckTypeList(el_textarea: HTMLTextAreaElement | null, cursor_pos_start: number): Promise<void> {
        let sub_string_textarea: string = el_textarea?.value.slice(cursor_pos_start-4, cursor_pos_start) || "";

        let DotList=(): void=>{
            // check string = "\n - "
            if(
                sub_string_textarea[0] == "\n" &&
                sub_string_textarea[1] == " "  &&   // to "\t"
                sub_string_textarea[2] == "-"  &&   // to "&#149;"
                sub_string_textarea[3] == " "
            )
            {
                if(el_textarea!=null){
                    el_textarea.value =
                        el_textarea.value.slice(0, cursor_pos_start-4) +
                        "\n" +
                        "   " +
                        ConvertToHTML("&#149;") +
                        " " +
                        el_textarea.value.slice(cursor_pos_start);
                    
                        el_textarea.selectionEnd = cursor_pos_start+2;
                }
            }
        }

        let NumberedList=(): void=>{

        }
        
        
        if(sub_string_textarea[2] == "-"){
            await DotList();
        }

        await NumberedList();
    }
    

    function ConvertToHTML(html_string: string): string {
        const doc: Document = new DOMParser().parseFromString(html_string, 'text/html');
        return doc.documentElement.textContent || "";
    }

</script>

<!-- ! on:paste={when input data}, you need to check the data for special characters! -->
<div class="record_field">
    {#each m_textarea_id as id_name}
        <textarea class="block_write" id={id_name}
            on:input={()=>HandleInputBlockWrite(id_name)}
            autofocus
        ></textarea>
    {/each}
</div>

<!--
A4:
    width: 794px,
    height: 1123px
Unicode UTF-8:
    &#149; - list(dot)
-->

<style lang="scss">
    /*==font==*/
    $absolute_font_family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    $absolute_font_weight: 300;
    /*==color==*/
        // text
    $text_white: rgb(229, 240, 255);
    $text_and_border_lite_blue: #5b7bac;
        // border
    // $border_lite_blue: #5C7198;
        // background
    $bg_dark_grey: #161b24;         // first choice
    $bg_more_dark_grey: #141820;    // first choice (more dark)
    $bg_dark_blue: #172030;         // second choice

    *{ //del
        // border: 2px solid #000;
    }

    .record_field{
        display: flex;
        flex-direction: column;
        align-items: center;
        width: 100%;
        height: 100%;
    }

    @mixin style_block_write{
        font-size: 16px;
        letter-spacing: 1.6px;
        font-family: $absolute_font_family;
        font-weight: $absolute_font_weight;
        background-color: $bg_more_dark_grey;
        width: 794px;
        height: 100%;
        padding: 30px 50px;
        margin: 5px;
        border: 0px;
        // border: 0px solid $text_and_border_lite_blue;
        border-radius: 5px;
        scrollbar-width: thin;
        scrollbar-gutter: stable;
    }

    /*===================*/
    /*=== Block Write ===*/
    /*===================*/

    .block_write{
        @include style_block_write;
        resize: none;
    }

    .block_write:focus{
        color: $text_white;
        outline: none;
    }
    
    .block_write::-webkit-scrollbar {
        width: 5px;
        height: 10px;
        
        &:hover{
            width: 10px;
        }
    }

    .block_write::-webkit-scrollbar-track {
        background: #00000000;
    }

    .block_write::-webkit-scrollbar-thumb {
        background-color: rgba(229, 240, 255, 0.2);
        border-radius: 3px;
        border: 0px;

        &:hover{
            background-color: rgba(229, 240, 255, 0.6);
        }
    }

</style>

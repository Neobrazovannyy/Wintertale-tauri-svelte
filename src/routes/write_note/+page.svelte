<script lang="ts">
    /*
    WARNING, ERROR, SHOULD
    MAKE_G - make global for personal settings
    */
    import { onMount } from "svelte";
    let g_block_write_El: HTMLDivElement;
    let g_block_console_El: HTMLInputElement;
    let g_local_rang_block_write: Range;

    /*==================================================*/
    /*=== Interface ====================================*/
    type RuEnMap ={
        [key: string]: string
    }

    //================================================================================================
    //================================================================== Setting global parameters ===
    onMount(()=>{ //LoadElementDOM
        window.addEventListener("keydown", CheckHotkey);
        g_block_write_El = document.querySelector(".block_write") as HTMLDivElement;
        g_block_console_El = document.querySelector(".block_console") as HTMLInputElement;
        // g_tree_block_write=SetTreeWalkerForBlockWrite();
        g_block_console_El.addEventListener("keydown", EnteredCommandIntBlockConsole);
    })

    // function SetTreeWalkerForBlockWrite(): TreeWalker{
    //     return document.createTreeWalker(
    //         g_block_write_El,
    //         NodeFilter.SHOW_ELEMENT,
    //         {
    //             acceptNode(node) {
    //                 if(node instanceof HTMLDivElement){
    //                     if(node.parentNode==g_block_write_El){
    //                         return NodeFilter.FILTER_ACCEPT
    //                     }
    //                     else return NodeFilter.FILTER_REJECT;
    //                 }
    //                 else{
    //                     return NodeFilter.FILTER_REJECT;
    //                 }
    //             }
    //         }
    //     );
    // }
    
    //===========================================================================================================
    //================================================================================ Hotkey & Walk Tree DOM === 

    // When you press the "hotkey", switch between blocks: block_write & block_console
    function CheckHotkey(event: KeyboardEvent){
        // Key down: CTRL + SHIFT;
        // if(event.ctrlKey && event.shiftKey)
        if(event.code === "Tab")
        {
            //just exists
            event.preventDefault();  //overrides the browser's default action
            event.stopPropagation(); //prevents the event from bubbling up to parent elements

            let block_active: HTMLDivElement | HTMLInputElement =document.activeElement as HTMLDivElement | HTMLInputElement;
            
            if(block_active!.className.slice(0, "block_write".length)=="block_write")
            {
                SetGVarLocalRangBlockWrite();
                g_block_console_El.focus();
            }
            else if(block_active!.className.slice(0, "block_console".length)=="block_console" || block_active==null){
                g_block_write_El.focus();
                SetCursorInBlockWrite();
            }
            else{
                g_block_write_El.focus();
                SetCursorInBlockWrite();
            }
            //TEST
            
            RouterConvertWordText(ConvertNodeToBoldFont)
        }
    }

    function SetGVarLocalRangBlockWrite(): void{
        const selection = window.getSelection();
        g_local_rang_block_write=selection!.getRangeAt(0);
    }

    function SetCursorInBlockWrite(): void{
        let range_block_write: Range=document.createRange();
        range_block_write.collapse(true);
        range_block_write.setStart(g_local_rang_block_write.startContainer, g_local_rang_block_write.startOffset);

        const update_sel = window.getSelection();
        if (update_sel) {
            update_sel.removeAllRanges();
            update_sel.addRange(range_block_write);
        }

    }

    function GetEdgeLineElement(edge_node: string): HTMLElement{
        let local_current_node: Node=g_local_rang_block_write.startContainer as HTMLElement;
        if(edge_node!="start" && edge_node=="end"){
            local_current_node=g_local_rang_block_write.endContainer as HTMLElement;
        }
        
        let local_current_element: HTMLElement=local_current_node.parentElement as HTMLElement;
        let previous_element: HTMLElement=local_current_element;
        let current_element: HTMLElement=local_current_element;
        let target_element: HTMLElement=local_current_element;
        
        if (local_current_element!.className.slice(0, "block_write".length)=="block_write"){
            return local_current_node as HTMLElement;
        }
        // else if(local_current_element!.className.slice(0, "record_field".length)=="record_field"){
        //     return null;
        // }

        // Finding a line element
        while(true)
        {
            current_element=previous_element.parentElement as HTMLElement;
            if(current_element!.className.slice(0, "block_write".length)=="block_write"){
                target_element=previous_element;
                break;
            }
            previous_element=current_element;
        }

        return target_element;
    }

    function GetNearbyNoEmptyElements(current_empty_element: HTMLElement, edge_node: string): HTMLElement | null{
        if(edge_node!="start" && edge_node!="end") return null;

        let walker: TreeWalker=document.createTreeWalker(
            g_block_write_El,
            NodeFilter.SHOW_ELEMENT,
            {
                acceptNode(node){
                    if(node.parentNode!=g_block_write_El){
                        return NodeFilter.FILTER_REJECT;
                    }
                    else{
                        return NodeFilter.FILTER_ACCEPT;
                    }
                }
            }
        );
    
        let FuncWhichWalker: Function;
        if(edge_node=="start"){
            FuncWhichWalker = () => walker.nextNode();
        }
        else{
            FuncWhichWalker = () => walker.previousNode();
        }

        walker.currentNode=current_empty_element as Node;
    
        let node: Node | null;
        while ((node=FuncWhichWalker())){
            if(node.textContent!=""){
                return node as HTMLElement;
            }
        }
        return null;
    }   
    
    //=========================================================================================================
    //================================================================== Enter Text Into Block Mini Console ===
    function EnteredCommandIntBlockConsole(event: KeyboardEvent): void {
        const ru_en_map: RuEnMap = {
            "а":"f","б":",","в":"d","г":"u", "д":"l","е":"t","ё":"`",
            "ж":";","з":"p","и":"b","й":"q", "к":"r","л":"k","м":"v",
            "н":"y","о":"j","п":"g","р":"h", "с":"c","т":"n","у":"e",
            "ф":"a","х":"[","ц":"w","ч":"x", "ш":"i","щ":"o",
            "ъ":"]","ы":"s","ь":"m","э":"\"","ю":".","я":"z"
        };
        setTimeout(()=>{
            g_block_console_El.value=g_block_console_El.value.replace(/[а-яё]/gi, char=> ru_en_map[char.toLowerCase()]);
        }, 100); //MAKE_G

        // if(event.ctrlKey && event.code==="KeyB"){
        //     event.preventDefault();
        //     event.stopPropagation();
        //     RouterConvertWordText(ConvertNodeToBoldFont)
        // }
        
        // if(event.ctrlKey && event.code==="KeyZ"){
        //     event.preventDefault();
        //     event.stopPropagation();
        // }

        if(event.code!=="Enter") return;
        if(g_block_write_El.textContent=="") return;

        event.preventDefault();
        event.stopPropagation();

        let console_content: string=g_block_console_El.value;
        if(console_content[0]=="h")
        {   /*----- <h1>...<h6> -----*/
            if(
                console_content.length === 2 &&
                ["/", "1","2","3","4","5","6"].includes(console_content[1])
            ){
                ConvertLineTextToHeader(console_content[1]);
            }
        }
        else if(console_content[0]=="b")
        {
            RouterConvertWordText(ConvertNodeToBoldFont)
        }

        // SelectFocusElementSetVarTreeAndCurPos("block_write");
        g_block_console_El.value=""
        g_block_write_El.focus();
        SetCursorInBlockWrite();
    }


    //========================================================================================
    //================================================================== Convert Line Text ===
    function ConvertLineTextToHeader(name_flag: string): void {
        var target_element: HTMLElement = GetEdgeLineElement("start");

        if(name_flag==="/"){
            target_element.style.fontSize = "16px";
            target_element.style.fontWeight = "300";
        }
        else if(name_flag==="1"){
            target_element.style.fontSize = "40px";
            target_element.style.fontWeight = "600";
        }
        else if(name_flag==="2"){
            target_element.style.fontSize = "32px";
            target_element.style.fontWeight = "600";
        }
        else if(name_flag==="3"){
            target_element.style.fontSize = "24px";
            target_element.style.fontWeight = "600";
        }
        else if(name_flag==="4"){
            target_element.style.fontSize = "20px";
            target_element.style.fontWeight = "600";
        }
        else if(name_flag==="5"){
            target_element.style.fontSize = "16px";
            target_element.style.fontWeight = "600";
        }
        else if(name_flag==="6"){
            target_element.style.fontSize = "14px";
            target_element.style.fontWeight = "600";
        }
    }


    //=========================================================================================
    //================================================================== Convert Words Text ===
    function RouterConvertWordText(ConvertNode: (node: Node)=>void ): void{
        let current_node: Node = g_local_rang_block_write.startContainer;
        let current_node_text: string | null =current_node.textContent;

        //Check for element content
        if(current_node_text!.trim().length==0 || current_node_text==null) return;
        l("current Node: " + current_node_text);

        let parent_element: Element=current_node.parentElement as Element;

        //If the "Current Node" consists of ONE word
        if(!current_node_text!.trim().includes(' ') && !parent_element.textContent!.trim().includes(' '))
        {
            l("The \"Current Node\" consists of ONE word");
            //If one word is in a node and the node is of the type: "span"
            if(parent_element instanceof HTMLSpanElement)
            {
                if(ConvertNode==ConvertNodeToBoldFont){
                    parent_element.style.fontWeight="600";
                }
            }
            //If one word is in a node and the node is of the type: "text"
            else
            {
                ConvertNode(current_node)
            }

        }
        //If the "Current Node" contains MULTIPLE words. A multi-word "Current Node" has THREE types:
        /*
            1) If the cursor is in an empty space;
            2) If the cursor is at a punctuation mark;
            3) If the cursor is at (on) a word.
        */
        else
        {
            l("The \"Current Node\" consists of MULTIPLE word");

            let pos_cursor: number=g_local_rang_block_write.startOffset;
            let current_text_node: Text=current_node as Text;
            let target_text_node: Text;

            /*----- 1) Checking if the cursor is between space characters -----*/
            let CheckSpaceAroundCursor=(pos: number)=>{
                return current_node_text[pos]==" " || current_node_text[pos]==undefined;
            };
            if(CheckSpaceAroundCursor(pos_cursor-1) && CheckSpaceAroundCursor(pos_cursor)) return;


            let list_punctuation_marks: string[]=[".", ",", ";", ":", "\\", "/", "!", "?"];

            /*----- 2) Punctuation marks selection test -----*/
            if(list_punctuation_marks.includes(current_node_text[pos_cursor-1]) && CheckSpaceAroundCursor(pos_cursor))
            {
                target_text_node=current_text_node.splitText(pos_cursor-1);
                target_text_node.splitText(1);
            }
            /*----- 3) If the selected node is not empty, not a punctuation mark, then we search for a word -----*/
            else
            {
                let pos_near_space: number=0;
    
                let is_first_word: boolean=true;
                for(let i=pos_cursor-1; i>=0; i--){
                    pos_near_space=i;
                    if(current_node_text[i]==" "){
                        is_first_word=false;
                        break;
                    }
                }
                if(!is_first_word) pos_near_space+=1;//skip index space
    
                target_text_node=current_text_node.splitText(pos_near_space); //cut off the beginning
    
                let target_content: string=target_text_node.textContent.trim();
                let check_has_space: number=target_content.indexOf(" ");
    
                //check is the last word
                if(check_has_space==-1){
                    if(list_punctuation_marks.includes(target_content[target_content.length-1])){
                        target_text_node.splitText(target_content.length-1);
                    }
                    else{
                        target_text_node.splitText(target_content.length);
                    }
                }
                else{
                    let target_word: string=target_content.substring(0,check_has_space);
    
                    if(list_punctuation_marks.includes(target_word[target_word.length-1])){
                        target_text_node.splitText(check_has_space-1);
                    }
                    else{
                        target_text_node.splitText(check_has_space);
                    }
                }
            }

            ConvertNode(target_text_node)
        }
    }

    function ConvertNodeToBoldFont(target_node: Node): void{
        let new_el: HTMLSpanElement=document.createElement("span");
        new_el.style.fontWeight="600";
        new_el.textContent=target_node.textContent;

        target_node.parentElement?.replaceChild(new_el, target_node);
    }

    function ConvertWordToBoldFont_Router(){
        // Check if the cursor is collapsed
        if(g_local_rang_block_write.collapsed){
        }

        let start_edge_line: HTMLElement = GetEdgeLineElement("start");
        
        // let end_edge_line: HTMLElement = GetEdgeLineElement("end");


        // check cursor or select. True=>"cursor"
        //start_edge_line==end_edge_line && g_local_rang_block_write.startOffset==g_local_rang_block_write.endOffset 0_0... g_local_rang_block_write.collapsed


        // let lol: HTMLElement | null =GetNearbyNoEmptyElements("start");
        // if()

        // g_local_rang_block_write
        // let start_current_element: HTMLElement =GetEdgeLineElement("start");
        // let end_current_element: HTMLElement =GetEdgeLineElement("end");

        // if (start_current_element==null){
        //     start_current_element=GetNearbyNoEmptyElements("start");
        // }
        // if(end_current_element==null){
        //     end_current_element=GetNearbyNoEmptyElements("end");
        // }



    }


    //=========================================================================================================================
    //================================================================================ Helper functions (can be safely removed)
    
    let l = function(variable: unknown): void{
        console.log(variable);
    };

    function LV(description: string="", variable: unknown=""): void {
        console.log(`${description}: ${variable}`);
    }

</script>



<!-- ! on:paste={when input data}, you need to check the data for special characters! -->
<div class="record_field">

    <div
        class="block_write"
        contenteditable="true" 
        autofocus
    >
        <!-- <div>
            I welcome you to a winter&rsquo;s fairy tale &#10052;
        </div> -->

        <!-- <div>Title 1</div><div>pop</div><div>kik</div><div>xcx</div><div><br></div><div>Title 2</div><div>loli</div><div>:)</div><div><br></div><div>Title 3</div><div>up &amp; down</div><div><br></div><div>@End</div> -->
        <div>Myths are ancient, timeless tales,</div><div><br></div><div>Of gods and heroes, monsters, and whales.</div><div><br></div><div>They <span style="text-decoration: underline;">tried to <span style="font-style: italic">explain</span> the <span style="font-style: italic">world's</span> creation</span>,</div><div><span style="font-weight: 600">Non-fiction</span></div><div><span style="font-weight: 600">And</span> <span style="font-weight: 600"> teach a <span style="font-weight: 800">les-son to </span> every</span> nation.</div><div>More <span style="font-weight: 600"><span style="font-weight: 600">than <span style="font-weight: 800">just</span> stories <span style="font-weight: 600">from</span></span></span> long ago,</div><div>They show us truths that we all know.</div><div><br></div><div><span style="font-weight: 600">Okak</span></div>
        <!-- <div>Myths are ancient, timeless tales,</div><div><br></div><div>Of gods and heroes, monsters, and whales.</div><div><br></div><div>They tried to explain the world's creation,</div><div><span style="font-weight: 600">teach</span></div><div>And teach a les-son to every nation.</div><div>More than just stories from long ago,</div><div>They show us truths that we all know.</div> -->
        <div>40empire</div>
        <div>Мороз <i><b>снежком</b></i> <span style="font-weight: 600">укутывал</span>: «Смотри, не <span> замерзай</span>!»</div>
        <div>Вы всегда <span>благородны, Неизменно <span>прекрасны, От </span> стремлений свободны</span>, К человеку бесстрастны.</div>


    </div>

    <input
        class="block_console"
        type="text"
        placeholder="TAB"
    >

</div>


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

    /*===================*/
    /*=== Block Write ===*/
    /*===================*/

    .block_write{
        color: $text_white;
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
        border-radius: 5px;
        /*-- Word Transfer --*/
        overflow-y: auto;
        overflow-x: hidden;
        white-space: pre-wrap;  /* transfer short word  */
        word-wrap: break-word;  /* transferring long words */
        /**/
        scrollbar-width: thin;
        scrollbar-gutter: stable;
        resize: none;

        &:focus{
            color: $text_white;
        }
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
    
    .block_console{
        position: absolute;
        background-color: $bg_more_dark_grey;
        color: $text_white;
        right: 0;
        bottom: 25px;
        height: 20px;
        border: 2px solid $text_and_border_lite_blue;
        pointer-events: none;

        &:focus{
            background-color: $text_and_border_lite_blue;
            outline: none;
            box-shadow: none;
        }
    }
</style>

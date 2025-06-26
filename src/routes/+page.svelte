<script lang="ts">
    import { SvelteDate } from 'svelte/reactivity';

    const formatter = new Intl.DateTimeFormat(undefined, {
        year: "numeric",
        month: "numeric",
        day: "numeric",
        hour: "numeric",
        minute: "numeric",
        second: "numeric",
        hour12: false,
	});

    let passwordQty = $state(5);
    let charactersQty = $state(16);
    let alphaNumeric = $state(true);
    let entelOneChars = $state(true);
    let createdPasswords: string[] = $state([]);
    let textMode = $state(false);

    interface csvFile {
        [key: string]: string ;
    }
    
    const passwordOptions = {
        alphaNumeric: "1234567890abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ",
        entelOneChars: "!#$%()*+-/:=?@[\]_{|}",
    };

    let generatePasswords = function(){
        if(!alphaNumeric && !entelOneChars){
            return alert("Debe seleccionar al menos un tipo de diccionario.");
        }

        if(passwordQty < 1 || passwordQty > 200){            
            return;
        }

        if(charactersQty < 1 || charactersQty > 128){            
            return;
        }

        createdPasswords = [];

        for(let i = 0; i < passwordQty; i++){
            createdPasswords.push(generatePassword());
        }
    }

    let generatePassword = function() {

        let passInfo = "";
        
        let passChars = [];

        if (alphaNumeric) {
            passInfo += passwordOptions.alphaNumeric;
            passChars.push(getRandomChar(passwordOptions.alphaNumeric));
        }

        if (entelOneChars) {
            passInfo += passwordOptions.entelOneChars;
            passChars.push(getRandomChar(passwordOptions.entelOneChars));
        }

        while(passChars.length < charactersQty) {
            passChars.push(getRandomChar(passInfo));
        };
        
        passChars = shuffleFisherYates(passChars);

        while(existsConsecutiveness(passChars)){
            passChars = shuffleFisherYates(passChars);
        }

        return passChars.join("");
    };

    function existsConsecutiveness(fromString: string[]){
        for(let i = 0; i < fromString.length; i++){
            if(fromString[i] === fromString[i + 1]){
                return true;
            }
        }
        return false;
    }

    function shuffleFisherYates(fromString: string[]){
        for(let i = fromString.length - 1; i > 0; i--){
            const swapIndex = randRange(i + 1);
            const temp: string = fromString[i];
            fromString[i] = fromString[swapIndex];
            fromString[swapIndex] = temp;
        };

        return fromString;
    }

    function getRandomChar(fromString: string){
        return fromString[randRange(fromString.length)];
    };

    function randRange(max: number) {
        const requestBytes = Math.ceil(Math.log2(max) / 8);

        if (!requestBytes) {
            return 0;
        };
        
        const maxNum = Math.pow(256, requestBytes);
        const ar = new Uint8Array(requestBytes);

        while (true) {
            window.crypto.getRandomValues(ar);

            let val = 0;
            for (let i = 0; i < requestBytes;i++) {
                val = (val << 8) + ar[i];
            };

            if (val < maxNum - maxNum % max) {
                return val % max;
            };
        };
    };

    const exportToCsv = (filename: string, rows: any[] , headers?: string[]): void => {
        if (!rows || !rows.length) {
            return;
        }
        const separator: string = ",";
        console.log(rows)
        const keys: string[] = Object.keys(rows);
        console.log(keys)
        let columHearders: string[];

        if (headers) {
            columHearders = headers;
        } else {
            columHearders = keys;
        }

        const csvContent =
            "sep=,\n" +
            columHearders.join(separator) +
            '\n' +
            rows.map(row => {
                
                let cell : any = row === null || row === undefined ? '' : row;

                cell = cell instanceof Date 
                    ? cell.toLocaleString()
                    : cell.toString().replace(/"/g, '""');

                if ((window.navigator as any).msSaveBlob) {
                    cell = cell.replace(/[^\x00-\x7F]/g, "");
                }
                if (cell.search(/("|,|\n)/g) >= 0) {
                    cell = `"${cell}"`;
                }
                return cell;
            }).join('\n');

        const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
        if ((window.navigator as any).msSaveBlob) { 
            (window.navigator as any).msSaveBlob(blob, filename);
        } else {
            const link = document.createElement('a');
            if (link.download !== undefined) {
                const url = URL.createObjectURL(blob);
                link.setAttribute('href', url);
                link.setAttribute('download', filename);
                link.style.visibility = 'hidden';
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
            }
        }
    }
</script>

<div class="max-w-4xl px-4 py-10 sm:px-6 lg:px-8 mx-auto">
    <div class="bg-white rounded-xl shadow-xs p-4 sm:p-7">
        <div class="flex justify-center items-center mb-5">
            <h1 class="text-xl">Generador de Contraseñas</h1>
        </div>
        <form>
            <div class="grid sm:grid-cols-12 gap-2 sm:gap-6">
                <div class="sm:col-span-6">
                    <label for="passwordQty" class="block mb-2 text-sm font-medium text-gray-900">Cantidad de Contraseñas</label>
                    <input type="number" bind:value={passwordQty} id="passwordQty" class="block w-full rounded-md bg-white px-3 py-1.5 text-base text-gray-900 -outline-offset-1 outline-gray-300 placeholder:text-gray-400 focus:outline-2 focus:-outline-offset-2 focus:outline-indigo-600 sm:text-sm/6" min="1" max="200" required />
                </div>
                <div class="sm:col-span-6">
                    <label for="charactersQty" class="block mb-2 text-sm font-medium text-gray-900">Cantidad de Caracteres</label>
                    <input type="number" bind:value={charactersQty} id="charactersQty" class="block w-full rounded-md bg-white px-3 py-1.5 text-base text-gray-900 -outline-offset-1 outline-gray-300 placeholder:text-gray-400 focus:outline-2 focus:-outline-offset-2 focus:outline-indigo-600 sm:text-sm/6" min="1" max="128" required />
                </div>

                <div class="flex flex-col items-center justify-center sm:col-span-4">
                    <label class="inline-flex items-center cursor-pointer">
                        <input type="checkbox" bind:checked={alphaNumeric} class="sr-only peer">
                        <div class="relative w-11 h-6 bg-gray-200 rounded-full peer dark:bg-gray-700 peer-checked:after:translate-x-full rtl:peer-checked:after:-translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:start-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:w-5 after:h-5 after:transition-all dark:border-gray-600 peer-checked:bg-blue-600 dark:peer-checked:bg-blue-600"></div>
                        <span class="ms-3 text-sm font-medium text-gray-900">Alfanumerico</span>
                    </label>
                </div>

                <div class="flex flex-col items-center justify-center sm:col-span-4">
                    <label class="inline-flex items-center cursor-pointer">
                        <input type="checkbox" bind:checked={entelOneChars} class="sr-only peer">
                        <div class="relative w-11 h-6 bg-gray-200 rounded-full peer dark:bg-gray-700 peer-checked:after:translate-x-full rtl:peer-checked:after:-translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:start-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:w-5 after:h-5 after:transition-all dark:border-gray-600 peer-checked:bg-blue-600 dark:peer-checked:bg-blue-600"></div>
                        <span class="ms-3 text-sm font-medium text-gray-900">Simbolos (Entel One)</span>
                    </label>
                </div>
                
                <div class="flex flex-col items-center justify-center sm:col-span-4">
                    <label class="inline-flex items-center cursor-pointer">
                        <input type="checkbox" bind:checked={textMode} class="sr-only peer">
                        <div class="relative w-11 h-6 bg-gray-200 rounded-full peer dark:bg-gray-700 peer-checked:after:translate-x-full rtl:peer-checked:after:-translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:start-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:w-5 after:h-5 after:transition-all dark:border-gray-600 peer-checked:bg-blue-600 dark:peer-checked:bg-blue-600"></div>
                        <span class="ms-3 text-sm font-medium text-gray-900">Modo Texto</span>
                    </label>
                </div>
                
                <div class="flex flex-col items-center justify-center sm:col-span-6">
                    <button id="generate" onclick={generatePasswords} class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm w-full sm:w-auto px-5 py-2.5 text-center">
                        Generar
                    </button>
                </div>
            
                <div class="flex flex-col items-center justify-center sm:col-span-6">
                    <button id="downloadCSV" disabled='{createdPasswords.length === 0}' onclick={() => exportToCsv(formatter.format(new SvelteDate()), createdPasswords, ['Passwords'])} class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm w-full sm:w-auto px-5 py-2.5 text-center disabled:bg-gray-50 disabled:text-gray-500 disabled:shadow-none">
                        Descargar CSV
                    </button>
                </div>

            </div>
        </form>
        <div id="createdPasswords" class="mt-5">
            {#if textMode}
                <div class="sm:col-span-2">
                    <p class="text-center">Listado de Contraseñas</p>
                </div>
                <div class="place-items-center mt-1">
                    <textarea name="createdPasswordsTextArea" id="createdPasswordsTextArea" class="block w-full rounded-md bg-gray-200 px-3 py-1.5 text-base text-gray-900 outline-1 -outline-offset-1 outline-gray-300 placeholder:text-gray-400 focus:outline-2 focus:-outline-offset-2 focus:outline-indigo-600 sm:text-sm/6" rows={createdPasswords.length} value={createdPasswords.join("\n")} readonly></textarea>
                </div>
            {:else}
                {#each createdPasswords as createdPassword, i}
                    <div class="grid sm:grid-cols-12 gap-2 sm:gap-6 mt-4 items-center justify-center">
                        <div class="sm:col-span-2">
                            <p class="text-center">{i+1}</p>
                        </div>
                        <div class="sm:col-span-10">
                            <input type="text" class="block w-full rounded-md bg-gray-200 px-3 py-1.5 text-base text-gray-900 -outline-offset-1 outline-gray-300 placeholder:text-gray-400 focus:outline-2 focus:-outline-offset-2 focus:outline-indigo-600 sm:text-sm/6 disabled:bg-gray-50 disabled:text-gray-500 disabled:shadow-none disabled:opacity-5" value={createdPassword} readonly>
                        </div>
                    </div>
                {/each}
            {/if}
        </div>
    </div>
    
</div>

<style lang="postcss">  
    @reference "tailwindcss";  
    
    :global(html) {    
        background-color: theme(--color-white);  
    }
</style>

<script>
    import axios from 'axios';
    import Koupit from '../components/Koupit.svelte';
    import {push, pop, replace} from 'svelte-spa-router';
    import { alertContent, uzivatel } from '../stores/stavy.js';
    import { nf,getImgUrl } from '../scripty/uzitecne.js';


    export let params;

    let loaded = false;
    let item = {};
    let itemDate;

    let selectedPic = 0;

    $: reFetch(params.id);

    function reFetch(id){
      axios({
        method: 'get',
        url: '/api/item/get',
        params: {
          id: id
        }
      }).then(res => {
        item = res.data;
        itemDate = new Date(item.release);
        loaded = true;
      }).catch(err => {
        alertContent.update(_ => err);
        replace('/');
      })
    }

    function galleryMove(dir){
      selectedPic += dir;
      if(selectedPic < 0){
        selectedPic = item.gallery.length - 1;
        return;
      }
      if(selectedPic > item.gallery.length - 1){
        selectedPic = 0;
      }
    }

</script>

<main>
  {#if loaded}
    <div class="ohraniceni">

        <div class="zpet"><button on:click={e => pop()}>Zpět</button></div>

        <div class="edit">
            {#if $uzivatel.perms}
                <a href={'/#/edit/' + item._id}><button></button></a>
            {/if}
        </div>
        
        <div class="nazev">{item.name}</div>

        <div class="vrch">

            <div class="obrazek">
                <img src={getImgUrl(item.thumbnail)} alt="error">
            </div>
        </div>
        <div class="spodek">
            <div class="cena">{nf(item.cost)} Kč</div>

            <div class="koupit"><Koupit  itemId={item._id}></Koupit></div>
        </div>
    </div>
    <div class="ohraniceni2">
        <div class="nadpis">Popis produktu</div>

        <div class="datum">Datum vydání: {`${itemDate.getDate()}.${itemDate.getMonth()}.${itemDate.getFullYear()}`}</div>

        <div class="text">{item.desc}</div>

        <div class="nadpis">Galerie obrázků</div>

        <div class="obrazky">
            <img src={getImgUrl(item.gallery[selectedPic])} alt="Žádné obrázky">
            <button class="control-left" on:click={_ => galleryMove(-1)}></button>
            <button class="control-right" on:click={_ => galleryMove(1)}></button>
        </div>
        <div class="cara"></div>
        <div class="rec">
            <div class="nadpis">Doporučené požadavky</div>
            <div class="prostredek">

                <div class="info">
                    <div>Operační systém: </div><br>
                    <div>DirectX: </div><br>
                    <div>Procesor: </div><br>
                    <div>Grafická Karta: </div><br>
                    <div>Paměť RAM: </div><br>
                    <div>Uložiště: </div>
                </div>
                <div class="vypisek">
                    <div>{item.os}</div><br>
                    <div>{item.dx}</div><br>
                    <div>{item.cpu}</div><br>
                    <div>{item.gpu}</div><br>
                    <div>{item.ram} GB</div><br>
                    <div>{item.size} GB</div>
                </div>
            </div>
        </div>

    </div>
  {/if}
</main>

<style>
    main {
        height: 100%;
        width: 100%;
        min-height: calc(100vh - 425px);
        position: relative;
        padding-bottom: 15px;
    }
    .ohraniceni {
        position: relative;
        max-width: 940px;
        height: 100%;
        min-height: 500px;
        background-color: var(--darkgrey);
        border-radius: 10px;
        margin: 0 auto;
        margin-top: 10px;
    }
    .ohraniceni2 {
        position: relative;
        max-width: 940px;
        height: 100%;
        min-height: 450px;
        background-color: var(--darkgrey);
        border-radius: 10px;
        margin: 20px auto 10px auto;
        padding-top: 5px;
    }
    .zpet{
        box-sizing: border-box;
        width: 50%;
        height: 50px;
        padding: 20px 0 0 40px;
        float: left;
    }
    .zpet button{
        height: 20px;
        width: 15px;
        padding-left: 15px;

        color: var(--text);
        font-size: 1em;

        background-image: url('/images/zpet.svg');
        background-repeat: no-repeat;
        background-position: center left;
        background-size: 13px;
    }
    .edit{
        position: relative;
        box-sizing: border-box;
        width: 50%;
        height: 50px;
        float: right;

    }
    .edit a button{
        position: absolute;
        right: 15px;
        top: 50%;
        transform: translateY(-50%);
        height: 25px;
        width: 25px;
        background-image: url('/images/settings.svg');
        background-repeat: no-repeat;
        background-position: center right;
        background-size: 25px;

    }
    .vrch, .spodek{
        position: relative;
        box-sizing: border-box;
    }
    .vrch{
        width: 100%;
        height: 100%;
        min-height: 350px;
        float: left;
    }
    .spodek{
        width: 100%;
        min-height: 35px;
        float: right;
    }
    .cena, .koupit{
        box-sizing: border-box;
        position: absolute;
    }
    .nazev{
        position: absolute;
        margin-top: 50px;
        width: 100%;
        height: 45px;
        color: var(--yellow);
        text-align: center;
        font-size: 2em;
    }
    .obrazek img{
        position: absolute;
        background-color: white;
        max-width: 256px;
        max-height: 320px;
        top: 5%;
        left: 50%;
        transform: translateX(-50%);
        margin-top: 40px;
    }
    .cena{
        left: 40.8%;
        transform: translateX(-50%);
        bottom: -45px;
        font-size: 1.3em;
    }
    .koupit{
        right: 30%;
        transform: translateX(-50%);
        bottom: -50px;
    }
    .nadpis{
        padding: 20px 40px;
        font-size: 1.7em;
    }
    .datum{
        margin: 0 0 20px 50px;
        font-size: 0.9em;
    }
    .text{
        box-sizing: border-box;
        border-bottom: solid 1px var(--text);

        font-size: 0.9em;
        line-height: 18px;

        padding: 0 15px 30px;
        margin: 0 35px;
    }

    .obrazky{
        position: relative;
        margin: 0 auto;
        padding: 0 0 30px;
        max-width: 800px;
        height: 449px;
        width: 100%;
    }

    .control-left, .control-right {
      position: absolute;
      top: 50%;
      width: 40px;
      height: 80px;
      background-repeat: no-repeat;
      background-position: center;
      background-size: 30px;
      background-image: url('/images/zpet.svg');
      background-color: rgba(0,0,0,0.3);
      transform: translateY(-50%);
    }
    .control-left {
      left: 0;
    }
    .control-right {
      right: 0;
      transform: translateY(-50%) rotate(180deg);
    }

    .obrazky img{
        max-width: 100%;
        max-height: 100%;
        display: block;
        margin: 0 auto;
    }
    .cara{
        width: 870px;
        box-sizing: border-box;
        margin: 0 auto;
        border-top: solid 1px var(--text);
    }
    .rec{
        position: relative;
        box-sizing: border-box;
        width: 100%;
        height: 100%;
        min-height: 250px;
    }
    .info, .vypisek{
        box-sizing: border-box;
    }
    .info{
        float: left;
        width: 20%;
        height: 160px;
        line-height: 12px;
        font-size: 1em;
        text-align:  left;
        padding: 10px 0 0 40px;
    }
    .vypisek{
        float: left;
        width: 75%;
        height: 160px;
        line-height: 12px;
        font-size: 1em;
        text-align: left;
        padding: 10px 0 0 10px;

    }

    @media only screen and (max-width: 1200px){
        .ohraniceni, .ohraniceni2{
            width: 700px;
        }
        .cena{
            left: 37%;
        }
        .koupit{
            right: 23%;
        }
        .cara{
            width: 630px;
        }
        .obrazky{
            max-width: 630px;
            height: 350px;
        }
        .control-left {
            left: 3px;
        }
        .control-right {
            right: 3px;

        }
        .info{
            width: 25%;

        }
        .vypisek{
            width: 70%;

        }
    }


    @media only screen and (max-width: 800px){
        .ohraniceni, .ohraniceni2{
            width: 400px;
        }
        .nazev{
            font-size: 1.5em;
            height: 30px;
        }
        .cena{
            left: 28%;
        }
        .koupit{
            right: 3%;
        }
        .nadpis{
            text-align: center;
            font-size: 1.3em;
        }
        .text{
            font-size: 0.7em;
            line-height: 15px;
            padding: 0 0 30px;
            margin: 0 35px;
        }
        .datum{
            margin: 0 0 15px 35px;
            font-size: 0.7em;
        }
        .info{
            left: 35px;
            height: 100px;
            width: 100px;
            line-height: 8px;
            font-size: 0.6em;
        }
        .obrazky{
            max-width: 330px;
            height: 180px;
        }
        .control-left {
            left: 5px;
            width: 40px;
            height: 60px;
        }
        .control-right {
            right: 5px;
            width: 40px;
            height: 60px;
        }
        .cara{
            width: 330px;
        }
        .rec{
            height: 100px;
            min-height: 200px;
        }
        .info{
            width: 30%;

        }
        .vypisek{
            left: 100px;
            height: 100px;
            width: 245px;
            line-height: 8px;
            font-size: 0.6em;
        }

    }
    
    @media only screen and (max-width: 440px){
        .ohraniceni, .ohraniceni2{
            width: 340px;
        }
        .nazev{
            font-size: 1.5em;
            height: 30px;
        }
        .cena{
            left: 28%;
        }
        .koupit{
            right: 0;
        }
        .nadpis{
            text-align: center;
            font-size: 1.3em;
        }
        .text{
            font-size: 0.7em;
            line-height: 15px;
            padding: 0 0 30px;
            margin: 0 15px;
        }
        .datum{
            margin: 0 0 15px 15px;
            font-size: 0.7em;
        }
        .info{
            left: 35px;
            height: 100px;
            width: 100px;
            line-height: 8px;
            font-size: 0.6em;
        }
        .obrazky{
            max-width: 330px;
            height: 180px;
        }

        .control-left {
            left: 5px;
            width: 40px;
            height: 60px;
        }
        .control-right {
            right: 5px;
            width: 40px;
            height: 60px;
        }
        .cara{
            width: 310px;
        }
        .rec{
            height: 100px;
            min-height: 200px;
        }
        .info{
            width: 30%;
            padding: 10px 0 0 15px;

        }
        .vypisek{
            left: 60px;
            height: 100px;
            width: 200px;
            line-height: 8px;
            font-size: 0.6em;
            padding: 10px 0 0
        }

    }
</style>

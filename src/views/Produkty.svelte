<script>
    import axios from 'axios';
    import RangeSlider from "svelte-range-slider-pips";
    import { onMount } from 'svelte';
    import { fade, slide, scale } from 'svelte/transition'
    import {parse} from 'qs';
    import {querystring} from 'svelte-spa-router';
    import { cats, scrollEvent } from '../stores/stavy.js'
    import PolozkaShelf from '../components/PolozkaShelf.svelte';

    export let params = {};
    let filtersState = false;
    let loaded = false;
    let items = [];
    let costLimit = [
      50,
      2000,
    ]
    let sortValue = 'soldUp';
    let fetchSkip = 0;
    let skipStop = false;
    let fetchAvalible = true;

    onMount(_ => {
      emptySkip();
    })

    $: emptySkip(params.cat,$querystring);
    $: reFetch(params.cat,parse($querystring).hledat,$scrollEvent);

    function emptySkip(){
      skipStop = false;
      fetchSkip = 0;
      items = [];
    }

    function fetchItems(query = {}, append){
      if(!fetchAvalible)return;
      fetchAvalible = false;
      axios({
        method: 'get',
        url: '/api/items/get',
        params: query
      }).then(res => {
        items = [...items, ...res.data];
        fetchSkip = items.length;
        if(res.data.length == 0)skipStop = true;
        fetchAvalible = true;
        loaded = true;
      }).catch(err => {
        //Špatně všechno
      })
    }

    function reFetch(cat,search,scroll = false){
      if(scroll){
        scrollEvent.update(_ => false)
        if(skipStop) return;
      }
      let query = {};
      if(cat)query.cat = cat;
      if(search)query.name = search;
      query.sort = sortValue;
      query.costMax = costLimit[1];
      query.costMin = costLimit[0];
      query.skip = fetchSkip;
      query.limit = 9;
      fetchItems(query,scroll);
    }

    function handleCostLimit(){
      emptySkip();
      reFetch(params.cat,parse($querystring).hledat);
    }

    function handleSort(e){
      sortValue = e.target.value;
      emptySkip();
      reFetch(params.cat,parse($querystring).hledat);
    }

    function handleFilters(){
      filtersState = !filtersState;
    }

    function handleScroll(){

    }

</script>
<main on:scroll={handleScroll}>
    <div class="ohraniceni1">
        {#if params.cat && $cats[params.cat]}
          <div class="nadpis">Kategorie: {$cats[params.cat]}</div>
        {/if}
        <div class="filtr">
            <button on:click={handleFilters}>Filtry</button>
        </div>
    </div>
    <div class="ohraniceni2" active={filtersState}>
        <div class="serazeni">
            <label for="seradit">Seřadit:</label>
            <select class="seradit" name="seradit" on:input={e => handleSort(e)}>
                <option value="costUp">Od nejdražšího</option>
                <option value="costDown">Od nejlevnějšího</option>
                <option value="soldUp" selected>Od nejprodávanějšího</option>
            </select>
        </div>

        <div class="cena">Cena:</div>
        <div class="sliderO">
            <RangeSlider id="slider" range pushy bind:values={costLimit} float max={2000} min={0} on:stop={handleCostLimit} pips step={50} suffix="Kč" />
        </div>

    </div><br>

    {#if loaded}
        <div class="sloupce">
            <div class="polozka">
                <div class="polozky">
                    {#each items as item,i}
                        <div in:scale={{duration:120 * i}}>
                            <PolozkaShelf details={item} />
                        </div>
                    {/each}
                </div>
            </div>

        </div>
    {/if}
</main>
<style>
    main{
        padding: 5px 0;
    }
    .ohraniceni1, .ohraniceni2{
        position: relative;
    }
    .ohraniceni1{
        width: 940px;
        height: 50px;
        background-color: var(--darkgrey);
        border-radius: 10px;
        margin: 0 auto;
        margin-bottom: 20px;
    }
    .ohraniceni2{
        display: none;
        margin: 0 auto;
        width: 940px;
        height: 120px;
        background-color: var(--darkgrey);
        border-radius: 10px;
        padding: 20px 30px;
        box-sizing: border-box;
    }
    .ohraniceni2[active="true"] {
      display: block;
    }
    .nadpis{
        position: absolute;
        font-size: 1.2em;
        line-height: 50px;
        left: 30px;
    }
    .filtr{
        position: absolute;
        float: right;
        background-color: var(--darkgrey);
        right: 30px;
        line-height: 50px;
    }
    .filtr button{
        line-height: 50px;
        color: var(--text);
        font-size: 1.2em;
    }
    .seradit{
        position: absolute;
        float: right;
        background-color: var(--darkgrey);
        left: 115px;
        line-height: 50px;
    }
    select {
        color: var(--text);
        text-align: center;
        text-align-last: center;
        outline: 0;
    }
    option {
        background-color: var(--darkgrey);
        text-align: center;
        text-align-last: center;
    }
    .serazeni{
        width: 100%;
    }
    .cena, .sliderO{
        margin-top: 20px;
    }
    .cena{
        line-height: 50px;
        height: 50px;
        width: 5%;
        float: left;
    }
    .sliderO{
        padding-top: 5px;
        height: 50px;
        width: 90%;
        float: left;
        margin-left: 25px;
    }

    .polozky{
        max-width: 940px;
        width: calc(100% - 80px);
        margin: 20px auto 0 auto;
        display: flex;
        flex-wrap: wrap;
        justify-content: space-around;
    }

    @media only screen and (max-width: 1200px){
        .ohraniceni1, .ohraniceni2{
            width: 760px;
            margin: 20px auto;
        }
        main{
            padding-bottom: 20px;
        }
        .seradit{
            left: 100px;
        }
    }
    @media only screen and (max-width: 800px){
        .ohraniceni1, .ohraniceni2{
            width: 400px;
            margin: 20px auto;
        }
        .cena{
            width: 15%;
        }
        .sliderO{
            width: 80%;
            margin-left: 10px;
        }
    }
    @media only screen and (max-width: 440px){
        .ohraniceni1, .ohraniceni2{
            width: 340px;
            margin: 10px auto;
        }
        .cena{
            width: 15%;
        }
        .sliderO{
            width: 80%;
            margin-left: 10px;
        }
    }



</style>

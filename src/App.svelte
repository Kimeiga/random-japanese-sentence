<script>
  let voice = window.speechSynthesis
    .getVoices()
    .filter((v) => v.lang.startsWith("ja"))[0];

  var decodeEntities = (function () {
    // this prevents any overhead from creating the object each time
    var element = document.createElement("div");

    function decodeHTMLEntities(str) {
      if (str && typeof str === "string") {
        // strip script/html tags
        str = str.replace(/<script[^>]*>([\S\s]*?)<\/script>/gim, "");
        str = str.replace(/<\/?\w(?:[^"'>]|"[^"]*"|'[^']*')*>/gim, "");
        element.innerHTML = str;
        str = element.textContent;
        element.textContent = "";
      }

      return str;
    }

    return decodeHTMLEntities;
  })();

  // import axios from "axios";
  // import pos from "pos";
  // import chunker from "pos-chunker";

  let simplified = true;

  // import cedict from "coupling-dict-japanese";
  // cedict.searchByjapanese("世界", (words) => {
  //   console.log(words);
  // });

  // import RakutenMA from "rakutenma";
  // import * as model_zh from "./model_zh.json";
  // import chardic from "./zh_chardic";
  // import * as HanZenKaku from "./hanzenkaku";

  // let rma_zh = new RakutenMA(model_zh);
  // rma_zh.featset = RakutenMA.default_featset_zh;
  // rma_zh.hash_func = RakutenMA.create_hash_func(15);
  // rma_zh.ctype_func = RakutenMA.create_ctype_chardic_func(chardic);

  // let tokens = rma_zh.tokenize(
  //   HanZenKaku.f_hs2fs(HanZenKaku.f_hw2fw(HanZenKaku.f_h2z("我是中国人。")))
  // );

  // if (tokens) console.log(RakutenMA.tokens2string(tokens));

  // let rma = new RakutenMA(model);
  // rma.featset = RakutenMA.default_featset_zh;
  // rma.hash_func = RakutenMA.create_hash_func(15);

  // console.log(rma.tokenize(HanZenKaku.hs2fs(HanZenKaku.hw2fw(HanZenKaku.h2z("我是中国人。")))));

  // import japanesetokenizer from "japanese-tokenizer";
  // let tokenize = japanesetokenizer.loadFile("./cedict_ts.u8");

  // console.log(JSON.stringify(tokenize("我是中国人。"), null, "  "));
  // console.log(JSON.stringify(tokenize("我是中國人。"), null, "  "));

  // rma = new RakutenMA(model);
  // rma.featset = RakutenMA.default_featset_ja;
  // rma.hash_func = RakutenMA.create_hash_func(15);

  function makeid(length) {
    let result = "";
    let characters =
      "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
    let charactersLength = characters.length;
    for (let i = 0; i < length; i++) {
      result += characters.charAt(Math.floor(Math.random() * charactersLength));
    }
    return result;
  }
  let translationText;
  let japaneseText;
  let traditionaljapaneseText;
  let pinyinTextList;
  $: rubyTexts = [];

  // const input = "[君|きみ]の[成功|せい|こう]は[一生懸命|いっ|しょう|けん|めい][勉強|べん|きょう]した。";

  function extractFurigana(input) {
    const regex = /\[([^\]|]+)\|([^\]]+)\]/g;
    const result = [];

    let match;
    let lastIndex = 0;

    while ((match = regex.exec(input)) !== null) {
      const [fullMatch, kanji, readings] = match;
      const furigana = readings.replace(/\|/g, "");

      // Add the text before the current match
      if (lastIndex < match.index) {
        const noReadingText = input.slice(lastIndex, match.index);
        result.push({ char: noReadingText, reading: "" });
      }

      // Add the current match
      result.push({ char: kanji, reading: furigana });

      lastIndex = regex.lastIndex;
    }

    // Add the remaining text after the last match
    if (lastIndex < input.length) {
      const noReadingText = input.slice(lastIndex);
      result.push({ char: noReadingText, reading: "" });
    }

    return result;
  }

  function SetBackgroundImage() {
    // let words = new pos.Lexer().lex(translationText);
    // let tags = new pos.Tagger()
    //   .tag(words)
    //   .map(function (tag) {
    //     return tag[0] + "/" + tag[1];
    //   })
    //   .join(" ");

    // // debugger;
    // let nouns = chunker.chunk(tags, "[{ tag: NN|NNP|VBG }]");
    // // console.log(nouns);
    // let extractedNouns = [];
    // let nouns2 = [...nouns.matchAll(/\{([^}]+)\}/g)];
    // // console.log(nouns2);

    // for (let n of nouns2) {
    //   // console.log(n);
    //   extractedNouns.push(n[1].match(/(.*?)\//)[1]);
    // }

    // console.log("extractedNouns");
    // console.log(extractedNouns);

    // remove short words
    let extractedWords = translationText.split(" ").filter((x) => x.length > 4);

    if (extractedWords.length == 0) {
      extractedWords = translationText.split(" ").filter((x) => x.length > 3);
    }

    if (extractedWords.length == 0) {
      extractedWords = translationText.split(" ").filter((x) => x.length > 2);
    }

    // remove words with apostrophe's like "didn't"
    let noApostrophes = extractedWords.filter((x) => x.indexOf("'") == -1);
    if (noApostrophes) extractedWords = noApostrophes;

    console.log("extractedWords");
    console.log(extractedWords);

    let backgroundImageURL = `url("https://source.unsplash.com/random/?${extractedWords.join()}")`;

    // console.log(backgroundImageURL);

    document.body.style.backgroundImage = backgroundImageURL;
  }

  // function SetBackgroundImage() {
  //   let words = new pos.Lexer().lex(translationText);
  //   let tags = new pos.Tagger()
  //     .tag(words)
  //     .map(function (tag) {
  //       return tag[0] + "/" + tag[1];
  //     })
  //     .join(" ");

  //   // debugger;
  //   let nouns = chunker.chunk(tags, "[{ tag: NN|NNP|VBG }]");
  //   // console.log(nouns);
  //   let extractedNouns = [];
  //   let nouns2 = [...nouns.matchAll(/\{([^}]+)\}/g)];
  //   // console.log(nouns2);

  //   for (let n of nouns2) {
  //     // console.log(n);
  //     extractedNouns.push(n[1].match(/(.*?)\//)[1]);
  //   }

  //   // console.log("extractedNouns");
  //   // console.log(extractedNouns);

  //   let backgroundImageURL = `url("https://source.unsplash.com/random/?${extractedNouns.join()}")`;

  //   // console.log(backgroundImageURL);

  //   document.body.style.backgroundImage = backgroundImageURL;
  // }

  (async () => {
    const seed = makeid(4);
    console.log(seed);
    let a = await fetch(
      "https://tatoeba.elnu.com/?from=jpn&orphans=no&sort=random&to=eng&trans_filter=limit&trans_to=eng&unapproved=no&limit=1&rand_seed=" +
        // "https://dev.tatoeba.org/en/api_v0/search?from=cmn&orphans=no&sort=random&to=eng&trans_filter=limit&trans_to=eng&unapproved=no&rand_seed=" +
        // seed
        "cWbc"
      // WUHp sex
      // "h9ZP"
      // "8hQt"
      // "Po2n"
      // "0SbA"
      // "J73O"
      // "TxWJ"
      // "PSlA"
      // "JnkI"
    ).then((response) => response.json());

    // TODO: Cvy4

    let r = a.results[0];

    console.log(r);

    japaneseText = r.transcriptions[0].text;
    traditionaljapaneseText = r.text;

    const furiganaArray = extractFurigana(japaneseText);
    rubyTexts = furiganaArray;

    // change fullwidth final punctuation to halfwidth for better centering
    if (japaneseText[japaneseText.length - 1] == "。") {
      japaneseText = japaneseText.substring(0, japaneseText.length - 1) + "｡";
    } else if (japaneseText[japaneseText.length - 1] == "？") {
      japaneseText = japaneseText.substring(0, japaneseText.length - 1) + "?";
    }

    if (traditionaljapaneseText[traditionaljapaneseText.length - 1] == "。") {
      traditionaljapaneseText =
        traditionaljapaneseText.substring(
          0,
          traditionaljapaneseText.length - 1
        ) + "｡";
    } else if (
      traditionaljapaneseText[traditionaljapaneseText.length - 1] == "？"
    ) {
      traditionaljapaneseText =
        traditionaljapaneseText.substring(
          0,
          traditionaljapaneseText.length - 1
        ) + "?";
    }

    // do full width ! too

    let translation = r.translations.filter((a) => a.length > 0);
    translationText = translation[0][0].text;

    document.getElementById("enText").innerText = translationText;

    getIndividualWordTranslations();

    //   const fetchNames = async () => {
    //   try {
    //     const res = await Promise.all([
    //       axios.get("https://japanese-dictionary.azurewebsites.net/" + rubyText.chars),
    //       axios.get("./names-mid.json"),
    //       axios.get("./names-old.json")
    //     ]);
    //     const data = res.map((res) => res.data);
    //     console.log(data.flat());
    //   } catch {
    //     throw Error("Promise failed");
    //   }
    // };

    // for (let rubyText of rubyTexts) {
    //   if (rubyText.chars == "?" || rubyText.chars == "｡") {
    //     return;
    //   }
    //   let res = await (
    //     await axios.get(
    //       "https://japanese-dictionary.azurewebsites.net/" + rubyText.chars
    //     )
    //   ).data;
    //   // console.log(r);
    //   console.log(res);

    //   console.log(res.length && res[0].definitions);
    //   let r;

    //   // filter out surnames
    //   let res2 = res.filter((r) => !/[A-Z]/.test(r.pronunciation));
    //   res = res2.length ? res2 : res;

    //   // filter out variants?
    //   let res3 = res.filter((r) => !/variant/.test(r.definitions));
    //   console.log(res3);
    //   res = res3.length ? res3 : res;

    //   if (res.length) {
    //     let first = res[0].definitions.match(/^[^;]*/);
    //     if (first) {
    //       r = first[0];

    //       console.log(r);

    //       // you (informal, as opposed to courteous 您[nin2]) -> you
    //       let r2 = r.replace(/\([^()]*\)/g, "").trim();
    //       r = r2.length ? r2 : r;
    //       console.log("r");
    //       console.log(r);
    //     } else {
    //       // no ;
    //       r = res[0].definitions;

    //       // you (informal, as opposed to courteous 您[nin2]) -> you
    //       let r2 = r.replace(/\([^()]*\)/g, "").trim();
    //       r = r2.length ? r2 : r;
    //       console.log("r");
    //       console.log(r);
    //     }
    //   }

    //   rubyText.def = r;
    //   rubyTexts = rubyTexts;
    //   // console.log(rubyTexts);
    // }

    // if we are still missing a final ? we got to add it
    // if (rubyTexts[rubyTexts.length - 1].text[rubyTexts[rubyTexts.length - 1].text.length - 1] == "?"){
    //   if (rubyTexts[rubyTexts.length - 1].chars != "？"){
    //     rubyTexts = [...rubyTexts, console.]
    //   }
    // }

    // get translations for each word!!

    // for (let r of rubyTexts) {

    // }

    // let a = 0;
    // let t = japaneseText;
    // for (let i = 0; i < syllableArray.length; i++) {
    //   if(t[a] == "，"){

    //   }
    //   else {
    //     const n = syllableArray[i];

    //     console.log(i, n, pinyinTextList[i], t.slice(a, a+n))
    //     a += n;
    //   }
    // }

    // for (let p of pinyinTextList) {
    //   const node = document.createElement("p");
    //   // const textnode = document.createTextNode(p);
    //   node.innerHTML = p;
    //   document.getElementById("pinyinTextDiv").appendChild(node);
    // }

    // console.log(rubyTexts);

    SetBackgroundImage();
  })();
  let msg = new SpeechSynthesisUtterance();

  function getIndividualWordTranslations() {
    // todo ignore ? in rubyText.chars
    // new more efficient way to get definitions with Promise.all!
    Promise.all(
      rubyTexts.map((rubyText) =>
        fetch("https://japanese-dictionary.azurewebsites.net/" + rubyText.chars)
          .then((response) => response.json())
          .catch((e) => console.error(e))
      )
    )
      // .then((res) => res.map((res) => (res ? res.data : [])))
      .then((texts) => {
        console.log("texts");
        console.log(texts);

        for (const [i, rubyText] of rubyTexts.entries()) {
          let res = texts[i];
          if (res.length == 0) {
            continue;
          }

          // filter out surnames
          let res2 = res.filter((r) => !/[A-Z]/.test(r.pronunciation));
          res = res2.length ? res2 : res;

          // filter out variants?
          let res3 = res.filter((r) => !/variant/.test(r.definitions));
          console.log(res3);
          res = res3.length ? res3 : res;

          // try to filter out definitions that use a different pronunciation
          let res4 = res.filter((r) => r.pronunciation == rubyText.text);
          console.log(res4);
          res = res4.length ? res4 : res;
          let r;

          // Todo: UUTa
          // 得
          // structural particle: used after a verb , linking it to following phrase indicating effect, degree, possibility etc

          if (res.length) {
            let first = res[0].definitions.match(/^[^;]*/);
            if (first) {
              r = first[0];

              console.log(r);

              // you (informal, as opposed to courteous 您[nin2]) -> you
              let r2 = r.replace(/\([^()]*\)/g, "").trim();
              r = r2.length ? r2 : r;
              console.log("r");
              console.log(r);
            } else {
              // no ;
              r = res[0].definitions;

              // you (informal, as opposed to courteous 您[nin2]) -> you
              let r2 = r.replace(/\([^()]*\)/g, "").trim();
              r = r2.length ? r2 : r;
              console.log("r");
              console.log(r);
            }
          }

          rubyText.def = r;
          rubyTexts = rubyTexts;
        }
      });
  }
</script>

<!-- 
<svelte:head>
  <script type="text/javascript" src="./rakutenma.js" charset="UTF-8"></script>
  <script
    type="text/javascript"
    src="./model_zh.js"
    charset="UTF-8"
    on:load={initZh}></script>
  <script type="text/javascript" src="./zh_chardic.js" charset="UTF-8"></script>
  <script type="text/javascript" src="./hanzenkaku.js" charset="UTF-8"></script>
</svelte:head> -->

<!-- <main>
  <div>
    <a href="https://vitejs.dev" target="_blank"> 
      <img src="/vite.svg" class="logo" alt="Vite Logo" />
    </a>
    <a href="https://svelte.dev" target="_blank"> 
      <img src={svelteLogo} class="logo svelte" alt="Svelte Logo" />
    </a>
  </div>
  <h1>Vite + Svelte</h1>

  <div class="card">
    <Counter />
  </div>

  <p>
    Check out <a href="https://github.com/sveltejs/kit#readme" target="_blank">SvelteKit</a>, the official Svelte app framework powered by Vite!
  </p>

  <p class="read-the-docs">
    Click on the Vite and Svelte logos to learn more
  </p>
</main> -->
{#each rubyTexts as r}
  <ruby>{r.char}<rt>{r.reading}</rt></ruby>
{/each}

<div id="container">
  <!-- <p id="cmnText" />
  <div id="pinyinTextDiv" /> -->
  <!-- {#if pinyinTextList}
    {#each pinyinTextList as p}
      <p>{p}</p>
    {/each}
  {/if} -->
  <!-- <div id="ruby" /> -->
  <div id="ruby">
    <!-- {#if japaneseText && (japaneseText[japaneseText.length - 1] == "。" || japaneseText[japaneseText.length - 1] == "？")}
      <span>&ensp;</span>
    {/if} -->
    {#if rubyTexts.length > 0}
      {#each rubyTexts as r}
        <ruby class="main-ruby" style="ruby-position: under;">
          <div
            style="display: flex; flex-direction:column;align-items: center;"
          >
            <ruby style="ruby-position: over; ">
              {r.char}
              <rt>{r.reading}</rt>
            </ruby>

            <p
              style="font-size: calc(0.5rem + 0.45vw); word-wrap: break-word;width: fit-content; text-align: center; margin: 0;"
            >
              {r.def ?? ""}
            </p>
          </div>
        </ruby>
      {/each}
    {:else}
      <small>Generating japanese Sentence...</small>
    {/if}
  </div>
</div>

<!-- <p id="pinyinText" /> -->
<hr style="margin: 1rem" />
<p id="enText" style="margin: 1.5rem;" />

<button
  on:click={() => {
    msg.text = japaneseText;
    msg.lang = "zh-CN";
    msg.voice = voice;
    msg.rate = 0.8;
    window.speechSynthesis.speak(msg);
  }}>Speak sentence</button
>

<button
  style="margin-left: 0.5rem;"
  on:click={() => {
    navigator.clipboard.writeText(
      simplified ? japaneseText : traditionaljapaneseText
    );
  }}>Copy japanese Sentence</button
>

<button
  style="margin-left: 0.5rem;"
  on:click={() => {
    // console.log(japaneseText + " -> " + converterToTraditional(japaneseText));

    // japaneseText = converterToTraditional(japaneseText);

    simplified = !simplified;

    // if (simplified) {
    //   rubyTexts = rubyTexts.map((r) => ({
    //     text: r.text,
    //     chars: converterToTraditional(r.chars),
    //   }));
    //   simplified = false;
    // } else {
    //   rubyTexts = rubyTexts.map((r) => ({
    //     text: r.text,
    //     chars: converterToSimplified(r.chars),
    //   }));
    //   simplified = true;
    // }
  }}
>
  Change to {#if simplified}
    Traditional
  {:else}
    Simplified
  {/if} Characters
</button>
<br />
<br />

<!-- <style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style> -->
<style>
  #pinyinTextDiv {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    width: 100%;
    font-size: 1.8rem;
  }

  #pinyinTextDiv p {
    font-size: 1.8rem;
  }

  #container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  ruby {
    display: inline-flex;
    flex-direction: column;
    align-items: center;
  }

  ruby > rt {
    font-size: 0.5em;
  }

  ruby::before {
    content: "hi";
    display: block;
    font-size: 0.5em;
  }

  ruby > span {
    order: 2;
  }
</style>

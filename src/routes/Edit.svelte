<script>
import Markdown from "../widgets/Markdown.svelte"
import TextEditor from "../widgets/TextEditor.svelte"
import Textarea from "../widgets/Textarea.svelte"
import Gallery from "../widgets/Gallery.svelte"
export let params;
export let data;
let cat = false;
let id = false;
let item = false;
let index = false;
let collection = false;
let fields = {};
let loading = false;
let title = false;


$: if (params.cat && params.id) {
cat = params.cat;
id = params.id;
item = data[cat].filter(x => x.id == id)[0];
index = data[cat].findIndex(x => x.id == id);

if(cat=='posts'){
  collection = data.types.filter(x => x.slug == item.type)[0];
  title = collection.title;
}else{
  collection = {};
  collection.fields = [];
  title = 'category';
}

}

function save(){
  if(typeof data[cat][index].slug === 'undefined' || data[cat][index].slug == ''){
    slugifyTitle();
  }

  loading = true;
  let opts = {};
  opts.path = 'data.json';
  opts.type = 'json';
  opts.data = data;
  call_api('api/save', opts).then(function(res) {
    window.renderData(data)
    if (res.ok) {
      console.log('Saved');
      loading = false;
    } else {
      console.log('Error saving');
      setTimeout(function(){
          loading = false;
      }, 1000)

    }
  });

}

function slugifyTitle()
{
  let slug = data[cat][index].title.toString().toLowerCase()
    .replace(/\s+/g, '-')           // Replace spaces with -
    .replace(/[^\w\-]+/g, '')       // Remove all non-word chars
    .replace(/\-\-+/g, '-')         // Replace multiple - with single -
    .replace(/^-+/, '')             // Trim - from start of text
    .replace(/-+$/, '');            // Trim - from end of text

  data[cat][index].slug = slug+'-'+Math.floor(Math.random() * 999);
  data = data;
}

</script>

{#if collection}



<div class="row topnav">
<div class="col-6">
<h4>Edit {title}</h4>
</div>
<div class="col-6 text-right">
<button class="btn btn-dark btn-add" on:click="{save}">{#if loading}<span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span> {/if} &nbsp;Save</button>
</div>
</div>


<div class="content">


{#if cat=='categories'}
<b>Title</b>
<input type="text" class="form-control" bind:value="{data[cat][index].title}" />

{:else}

<div class="row">
<div class="col-md-8">

<b>Title</b>
<input type="text" class="form-control" bind:value="{data[cat][index].title}" />

</div>
<div class="col-md-4">

{#if cat !== 'categories'}
<b>Category</b>
<select bind:value="{data[cat][index].category}" class="form-control w-100">
{#each data.categories as cat}
<option value="{cat.slug}">{cat.title}</option>
{/each}
</select>
{/if}

</div>
</div>

{/if}

  {#each collection.fields as field}

{#if field.type !== 'cat'}
  <b>{field.title}</b>
{/if}

  {#if field.description}
  <div class="description">{field.description}</div>
  {/if}

  {#if field.type=='txt'}
  <input type="text" class="form-control" bind:value="{data[cat][index][field.title]}" />
  {/if}

  {#if field.type=='txta'}
  <Textarea bind:val={data[cat][index][field.title]} />
  {/if}

  {#if field.type=='mde'}
  <Markdown bind:key={field.title} bind:html={data[cat][index][field.title]} />
  {/if}

  {#if field.type=='rte'}
  <TextEditor bind:key={field.title} bind:html={data[cat][index][field.title]} />
  {/if}


  {#if field.type=='gal'}
  <Gallery bind:key={field.title} bind:item={data[cat][index]} />
  {/if}


  {/each}

</div>

{/if}

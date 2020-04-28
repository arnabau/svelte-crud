<script>
  import { onMount } from "svelte";
  import PostForm from "../components/PostForm.svelte";

  const apiBaseUrl =
    "https://ndb99xkpdk.execute-api.eu-west-2.amazonaws.com/dev";
  let posts = [];
  let editingPost = {
    body: "",
    title: "",
    id: null
  };

  let postLimit = 6;

  onMount(async () => {
    const res = await fetch(`${apiBaseUrl}/posts`);
    posts = await res.json();
  });

  const addPost = ({ detail: post }) => {
    if (posts.find(p => p.id === post.id)) {
      const index = posts.findIndex(p => p.id === post.id);
      let postsUpdate = posts;
      postsUpdate.splice(index, 1, post);
      posts = postsUpdate;
    } else {
      posts = [post, ...posts];
    }
    editingPost = {
      body: "",
      title: "",
      id: null
    };
  };

  const editPost = post => {
    editingPost = post;
  };

  const deletePost = id => {
    if (confirm("Are you sure?")) {
      fetch(`${apiBaseUrl}/post/${id}`, {
        method: "DELETE"
      })
        .then(res => {
          return res.json();
        })
        .then(() => {
          posts = posts.filter(p => p.id !== id);
        });
    }
  };

  const setLimit = () => {
    fetch(`${apiBaseUrl}/posts/${postLimit}`)
      .then(res => {
        return res.json();
      })
      .then(postsData => {
        posts = postsData;
      });
  };
</script>

<style>
  .card .card-content .card-title {
    margin-bottom: 0;
  }
</style>

<div class="row">
  <div class="col s6">
    <PostForm on:postCreated={addPost} {editingPost} />
  </div>
</div>

<div class="col s3">
  <p>Limit number of posts</p>
  <input type="number" bind:value={postLimit} />
  <button on:click={setLimit} class="waves-effect waves-light btn-small">
    Set
  </button>
</div>

<div class="row">
  {#if posts.length === 0}
    <h3>Loading posts...</h3>
  {:else}
    {#each posts as post}
      <div class="col s6">
        <div class="card">
          <div class="card-content">
            <p class="card-title">{post.title}</p>
            <p>{post.createdAt}</p>
            <p>{post.body}</p>
          </div>
          <div class="card-action">
            <a href="#" class="btn-small blue" on:click={() => editPost(post)}>
              Edit
            </a>
            <a
              href="#"
              class="btn-small red"
              on:click={() => deletePost(post.id)}>
              Delete
            </a>
          </div>
        </div>
      </div>
    {/each}
  {/if}
</div>

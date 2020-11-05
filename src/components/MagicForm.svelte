<script>
  import formatMessage from "format-message";
  import translations from "../locales";
  import { profile } from "../stores";
  import SearchField from "./SearchField.svelte";
  import Tags from "./Tags.svelte";
  import categories from "../data/categories";
  import Spinner from "./Spinner.svelte";
  import Api from "../services/api";

  let formRef;
  let lang;
  let loading = false;
  let isActiveSwitcher = false;
  let activeTags = [];
  let suggestions = categories.slice(0, 7);
  let searchQuery = "";
  let searchRef;
  export let message = "";
  export let handleDismiss;
  export let handleSuccess;

  $: lang = $profile.lang || "ru";
  $: {
    formatMessage.setup({
      missingTranslation: "ignore",
      translations,
      locale: lang,
    });
  }
  $: totalTagsCount = searchQuery.length
    ? suggestions.length
    : categories.length;
  $: suggestions = searchQuery.length
    ? categories.filter((tag) => tag.value.includes(searchQuery))
    : categories.slice(0, 7);

  const _handleSubmit = (e) => {
    doPost();
    e.preventDefault();
    if (handleSuccess) handleSuccess();
  };

  const _handleSearch = () => {
    console.log("_handleSearch", searchQuery);
  };

  const _handleSearchReset = () => {
    searchQuery = "";
    suggestions = categories.slice(0, 7);
    console.log("_handleSearchReset");
  };

  async function doPost() {
    const res = await Api.call("/domagic", {
      method: "POST",
      body: {
        options: activeTags.map((item) => item.value),
        spell: message,
      },
    });

    return res;
  }
</script>

<section class="magic-form-wrapper">
  <form bind:this="{formRef}" id="magicForm" class="magic-form">
    <div class="message-bubble">
      <div class="search-wrapper">
        <SearchField
          ref="{searchRef}"
          placeholder="{formatMessage('Search')}"
          bind:query="{searchQuery}"
          handleSubmit="{_handleSearch}"
          handleReset="{_handleSearchReset}"
          loading="{false}"
          lightTheme={true} />
      </div>

      <Tags
        items="{suggestions}"
        activeValues="{activeTags}"
        handleClick="{(item) => {
          activeTags = [...activeTags, item];
          suggestions = suggestions.filter((tag) => tag.value !== item.value);
          suggestions.unshift(item);
        }}"
        handleDismiss="{(item) => {
          activeTags = activeTags.filter((tag) => tag.value !== item.value);
          suggestions = suggestions.filter((tag) => tag.value !== item.value);
          const initialIndex = categories.findIndex((tag) => tag.value === item.value);
          suggestions.splice(initialIndex, 0, item);
        }}"
        handleMore="{() => {
          suggestions = [...suggestions, ...categories.slice(suggestions.length, suggestions.length + 7)];
        }}"
        color="#edeef3"
        textColor="#82829a"
        totalCount="{totalTagsCount}" />

      <textarea
        id="message"
        name="message"
        class:faded="{loading}"
        bind:value="{message}"
        placeholder="{formatMessage('Type a spell')}"></textarea>

      <div class="switcher">
        <input
          id="switcher"
          type="checkbox"
          bind:checked="{isActiveSwitcher}" />
        <label for="switcher"></label>
        <span>{formatMessage('ToS Agreement')}</span>
      </div>

      {#if loading}
        <div class="loader-wrapper">
          <Spinner />
        </div>
      {/if}
    </div>

    <div class="message-actions" class:hidden="{loading}">
      <button
        name="job_apply"
        id="job_apply"
        on:click="{_handleSubmit}"
        disabled="{!isActiveSwitcher || !message.length}">
        {formatMessage('Save')}
      </button>
      <span class="text-button" on:click="{handleDismiss}">
        {formatMessage('Cancel')}
      </span>
    </div>
  </form>
</section>

<style lang="scss">
  @import "../styles/theme";

  .loader-wrapper {
    display: flex;
    position: absolute;
    width: 100%;
    height: 100%;
    justify-content: center;
    align-items: center;
    top: 0;
    margin-left: -25px;
  }

  .faded {
    opacity: 0.25;
  }

  .hidden {
    opacity: 0;
  }

  .message-bubble {
    width: 100%;
    height: 100%;
    background: white;
    border-radius: 12px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.22);
    padding: 20px 25px;
    padding-bottom: 10px;
    margin-top: 10px;

    & textarea {
      margin-top: 20px;
      font-size: 15px;
      height: 170px;

      &::placeholder {
        color: $lightText;
        opacity: 0.7;
      }
    }
  }

  .message-actions {
    width: 100%;
    margin: 20px 0;
    text-align: right;

    & button {
      background: $accent;
      color: white;

      &:hover {
        background: $accentHover;
        box-shadow: none;
      }
      &:disabled {
        color: rgba(0, 0, 0, 0.3);
        background: rgba(0, 0, 0, 0.1);
        box-shadow: none;
      }
    }

    & span {
      margin-left: 20px;

      &.text-button {
        color: black;
        margin-left: 0;
        float: left;
      }
    }
  }

  .switcher {
    display: flex;
    align-items: center;
    position: relative;

    span {
      color: $lightText;
    }

    input[type="checkbox"] {
      position: absolute;
      top: 0;
      right: 0;
      height: 0;
      width: 0;
      visibility: hidden;
    }

    label {
      cursor: pointer;
      text-indent: -9999px;
      width: 30px;
      height: 16px;
      background: $lightText;
      display: block;
      border-radius: 15px;
      position: relative;
      margin-right: 10px;
    }

    label:after {
      content: "";
      position: absolute;
      top: 1px;
      left: 1px;
      width: 14px;
      height: 14px;
      background: #fff;
      border-radius: 9px;
      transition: 0.3s;
    }

    input:checked + label {
      background: $accent;
    }

    input:checked + label:after {
      left: calc(100% - 1px);
      transform: translateX(-100%);
    }

    label:active:after {
      width: 20px;
    }
  }
</style>

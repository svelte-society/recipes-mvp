# recipes-mvp

MVP repo to just start collecting recipes for [Svelte Community](https://github.com/sveltejs/community). The core inspiration is https://vuejs.org/v2/cookbook/.

All examples have some variation "with Svelte" appended to them in an attempt to optimize for searchability.

> NOTE: This is a new project! What you see below is an evolving work in progress. Please see [Project Goals](https://github.com/svelte-society/recipes-mvp/issues/9) for more. We actively need contributions of ideas and content!

# Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
<details>
<summary>Table of Contents</summary>

- [Build Setup Recipes](#build-setup-recipes)
  - [Using Svelte with other technologies (e.g. PostCSS, SCSS, TypeScript, and Babel)](#using-svelte-with-other-technologies-eg-postcss-scss-typescript-and-babel)
  - [Transpiling ES6 to ES5 for Legacy Browser (IE11) Support with Babel](#transpiling-es6-to-es5-for-legacy-browser-ie11-support-with-babel)
  - [Using Future JS Syntax in Svelte with Babel](#using-future-js-syntax-in-svelte-with-babel)
  - [Using TypeScript with Svelte](#using-typescript-with-svelte)
  - [Using PostCSS/SCSS with Svelte](#using-postcssscss-with-svelte)
  - [Writing Your Own Preprocessors](#writing-your-own-preprocessors)
  - [How to make a pre-processor that makes it possible to use PUG/Jade](#how-to-make-a-pre-processor-that-makes-it-possible-to-use-pugjade)
- [Svelte Language Fundamentals](#svelte-language-fundamentals)
  - [Reactivity](#reactivity)
- [Svelte Component Recipes](#svelte-component-recipes)
  - [Using Fetch to Consume APIs with Svelte](#using-fetch-to-consume-apis-with-svelte)
  - [Form Validation with Svelte](#form-validation-with-svelte)
  - [Client-Side Storage with Svelte](#client-side-storage-with-svelte)
- [Svelte Store Recipes](#svelte-store-recipes)
  - [Stores](#stores)
- [Svelte Action Recipes](#svelte-action-recipes)
  - [Creating Custom Scroll Actions with Svelte](#creating-custom-scroll-actions-with-svelte)
- [Svelte Transition Recipes](#svelte-transition-recipes)
- [Svelte App-Level Design Patterns](#svelte-app-level-design-patterns)
  - [Routing with Svelte](#routing-with-svelte)
  - [Authentication with Svelte](#authentication-with-svelte)
  - [Server Side Rendering](#server-side-rendering)
- [Svelte Performance Tips](#svelte-performance-tips)
  - [Avoiding Memory Leaks with Svelte](#avoiding-memory-leaks-with-svelte)
- [Testing and Debugging Svelte](#testing-and-debugging-svelte)
  - [Unit Testing Svelte Components](#unit-testing-svelte-components)
  - [Debugging Svelte Apps in VS Code](#debugging-svelte-apps-in-vs-code)
- [Publishing Svelte Components/Deploying Svelte Apps](#publishing-svelte-componentsdeploying-svelte-apps)
  - [Packaging Svelte Components for npm](#packaging-svelte-components-for-npm)
  - [Dockerize a Svelte App](#dockerize-a-svelte-app)
- [Special Usecase Walkthroughs](#special-usecase-walkthroughs)
  - [Editable SVG Icon Systems with Svelte](#editable-svg-icon-systems-with-svelte)
  - [Practical use of Svelte slots with Google Maps](#practical-use-of-svelte-slots-with-google-maps)
  - [Create a CMS-Powered Blog with Svelte](#create-a-cms-powered-blog-with-svelte)

</details>
<!-- END doctoc generated TOC please keep comment here to allow auto update -->

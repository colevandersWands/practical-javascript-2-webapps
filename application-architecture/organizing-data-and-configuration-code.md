# Organizing Data and Configuration Code
As we work on different parts of our applications, we often find that we have similar needs in different places. We might have specific formatting we want to remain consistent across templates, or system data that needs to be available to different components. We might find that we are making API requests to the same API in different components and we want to minimize the amount of repetitive code we are using. In each of these cases, what we absolutely want to avoid is duplicating the same data or logic in multiple locations.

As we saw in previous sections, we will make our applications easier to maintain, build, and enhance if we can adhere to a clean separation of concerns and utilize some common strategies for managing components that need to be reused throughout the application. In this section we will look at some techniques for organizing reusable pieces of our system.

## Common Data Techniques
A common case that often comes up is the need to store system-specific data that can be accessed from any component. This is often used for sets of static metadata that are specific to the project and which are not subject to change. These are usually considered "constants" in the system: They are not meant to be altered by the user or to change during the use of the application or site.

Since we can easily import objects into our components, all we need to do is make sure we have properly structured our data file to export an object. We can store these files wherever makes sense: a `/common/` directory in our `/src/` directory, a well-named file, or somewhere else within the project.

Here is an example of a data file that can be imported into a Vue.js component. Assume this code is in the file `/src/common/constants.js`.

```js
export default {
    'metadataProperty': 'Some value',
    'someChoices': [
        { name: 'Choice One', value: 1 },
        { name: 'Choice Two', value: 2 },
        { name: 'Choice Three', value: 3 }
    ]
}
```
We could make use of this data object by importing it into a Vue.js component:

```html
<script>
import SystemData from `@/common/constants.js`;

export default {
  name: 'demoComponent',
  data () {
    return {
      formOptions: SystemData.someChoices
    }
  }
}
</script>
```

As we've seen in previous projects and examples, we can import the content of `constants.js` with the name `SystemData` inside our component. We can then use that data however we would like within our component logic. In this case, we have set the component's `formOptions` value to the `SystemData.someChoices` array. This could be used to provide a form input with consistent choices and formatting across the entire application.

This technique also shows the fundamental principle at play in the following examples. First we create a `.js` file that contains some object. We must make sure that object is properly "exported" with the `export` command. Then, we can import that object wherever we need it. This is fundamentally the same process we use to accomplish the other techniques for organizing information in our Vue.js applications.

## Common Configuration Techniques
Previously in this book we explored using the Axios module to perform HTTP requests to API endpoints. This is a powerful tool, and on a modern web project we might use several different API services that are all controlled by our backend systems. It is very common for developers to consume their own API services to build multiple frontends (e.g. mobile app and website). 

When using multiple endpoints on the same API service provider, it is often necessary to provide basic authentication information, to complete some sort of authentication handshake, or to otherwise use a common configuration. If we are making API calls from multiple components, we might find that we've duplicated this common data and logic several times throughout our application. We can refactor those API calls to use the same base instance.

This approach can work in many situations with JavaScript modules that rely on some form of common configuration. Let's take a look at an Axios example to get a better idea of what this looks like. The following code is stored in the file `/src/common/api.js`.

```js
import axios from 'axios';

export const API = axios.create({
  baseURL: `http://jsonplaceholder.typicode.com/`,
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  }
})
```
In this example, we are imagining that the `jsonplaceholder` API uses basic HTTP Auth for authorization. Obviously, we would prefer not to repeat this configuration in every single `.vue` file where we make a call to a different API endpoint. As we may remember from our previous work with `jsonplaceholder`, there are several endpoints (`posts`, `users`, etc.) and if we were building a full app we would use each of those endpoints.

Now that we have our basic API configuration abstracted into a standalone file, we can use it in another component. Let's imagine we have a component called `Posts.vue` that is pulling in the posts:

```html
<script>
import API from '@/common/api.js';

export default {
  data() {
    return {
      posts: [],
      errors: []
    }
  },
  created() {
    API.get(`posts`)
    .then(response => {
      this.posts = response.data
    })
    .catch(error => {
      this.errors.push(error)
    })
  }
}
</script>
```
Notice that in this example we don't need to specify anything beyond the endpoint path (`posts`). Our API call is much smaller than in previous examples. And if we used a different endpoint to get the photos for a post, the API call would look something like this:

```html
<script>
import API from '@/common/api.js';

export default {
  data() {
    return {
      photos: [],
      errors: []
    }
  },
  created() {
    API.get(`photos`, {
      params: {
        albumId: this.albumId
      }
    })
    .then(response => {
      this.photos = response.data
    })
    .catch(error => {
      this.errors.push(error)
    })
  },
  props: ['albumId']
}
</script>
```
In this example, we have a component that expects to receive an `albumId` property. This component is probably a photo gallery component that needs to receive data about the photos in this "album" and then display them. The API request is once again formed by importing the `API` object, and the URL for the endpoint is `photos`. Some query string parameters are added to the request (setting the `albumId` value), but otherwise the request looks the same as the previous request. And once again we have not duplicated the basic configuration information.

Not only is this a cleaner way of using the the same API service in multiple components, but it also opens the door for us to provide a mechanism to switch between a "production" and "development" API server. Now that our configuration is abstracted into a single location, we could enhance that configuration to properly alter which API server the application should contact. This is a very common use case for developers, who must often work with new functionality or data that is unavailable on the production API service.

## Common Filters and Methods

When working with components, we often find ourselves performing the same tasks over and over. For example, formatting dates, text, or monetary values is a common need in our templates. This formatting is best accomplished with a "filter", which can be applied to the output of a value in the template. Since most of the logic within a component is packaged as a JavaScript object, it is easy to define objects that can be imported and used within multiple components.

Let's consider the example of a filter that capitalizes a String value. This is often needed when displaying user data in a template because we cannot be sure the user themselves capitalized the text. We can follow the same patterns we used above to accomplish this goal.

First, let's make a file called `/src/common/filters.js`:

```js
export default {
  capitalize: function (value) {
    if (!value) return ''
    value = value.toString()
    return value.charAt(0).toUpperCase() + value.slice(1)
  }
}
```
This file defines an object that has one property: `capitalize`. That property is a function, that expects an argument called `value` and returns a modified form of the `value`. (This is the standard format for a Vue.js filter.)

We can use this filter in a component by importing it in the component logic and then using it in the template.

```html
<template>
  <div class="item">
    <h2>{{ name }}</h2>
    <p>Price: ${{ price }}</p>
    <button v-on:click="addToShoppingCart">Add to Cart</button>
  </div>
</template>

<script>
import CommonFilters from `@/common/filters.js`;

export default {
  name: 'item',
  data () {
    return {

    }
  },
  props: [
    'name',
    'price'
  ],
  methods: {
    addToShoppingCart: function () {
      console.log(`Adding ${ this.name } to cart.`);
      this.$emit('addedItem');
    }
  },
  filters: CommonFilters
}
</script>
```









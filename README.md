# netsuite-vue-boilerplate [![Build Status](https://travis-ci.org/vinikira/netsuite-vue-boilerplate.svg?branch=master)](https://travis-ci.org/vinikira/netsuite-vue-boilerplate)
A vuejs boilerplate for netsuite, that allow build all application in one file html for put in your File Cabinet. 
## Getting started
Clone this repository and install dependenciess.
```sh
 git clone https://github.com/vinikira/netsuite-vue-boilerplate && cd netsuite-vue-boilerplate && npm install
```
## Developing
Develope your application like any [VueJS](https://github.com/vuejs/vue) application.

Probably you'll want to integrate your application with a RESTlet script, in these cases you can add a proxy to package.json:
```json
...
  "proxy": "https://your-account.restlets.api.netsuite.com"
}
```
And then you can prefixes authentication header to you http library (remember to make it only in development envioriment) on src/index.js, following is an example using [Axios](https://github.com/axios/axios):

```javascript
if (process.env.NODE_ENV === `development`) {
  const { NSEMAIL, NSPASSWORD, NSACCOUNT } = process.env

  axios.defaults.headers = {
    'Content-Type': 'application/json;charset=utf-8',
    'User-Agent-x': 'SuiteScript-Call',
    'Authorization': `NLAuth nlauth_account = ${NSACCOUNT}, nlauth_email = ${NSACCOUNT}, nlauth_signature = ${NSPASSWORD}`
  }
}
```

After doing this you can call any restlet locally sending only path string:

```javascript
axios.get('/app/site/hosting/restlet.nl?script=script-id&deploy=restlet-deploy') 
```

This will facilitate your development, because you not need to build and upload the index.html always you make a change in your application.

## Building
Use this command to generate a production build:
```sh
npm run build
```
This command will generate a folder (your-project/dist) that contains the index.html, bundle.js.map and the bundle.js files, all minified.

## Deploying
Copy build/index.html (ignore bundle.js and bundle.js.map because it is inside of index.html) to your NetSuite SuiteScript folder, then is just click in html file on netsuite that a popup will be open with your application.  

You can also use a Suitelet Script to serve your application.
## Examples
### Hello world application
src/App.vue - App Vue component.
```vue
<template>
  <div id="root">
    <h1>Hello World!</h1>
    <p>{{ msg }}</p>
  </div>
</template>

<script>
  export default {
      name: 'root',
      data () {
          return {
              msg: 'Hello NetSuite!'
          }
      }
  }
</script>

<style>
</style>
```
src/index.js - File that will bootstrap application.
```javascript
import Vue from 'vue'
import App from './App.vue'

new Vue({
    el: '#app',
    render: h => h(App)
})
```
src/index.html - this file is necessary to build an minified .html and for indicate what <div> your application will be stored.
```html
<!DOCTYPE html>
<html lang="pt-br">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```
### Suitelet to serve your application
```javascript
  /**
 * @NApiVersion 2.x
 * @NScriptType Suitelet
 */
define(['N/ui/serverWidget', 'N/file'],
  function (ui, file) {
    function onRequest(context) {
      'use strict'
      if (context.request.method === 'GET') {

        const form = ui.createForm({
          title: 'Your application title'
        })

        const contentHTML = form.addField({
          id: 'content',
          type: ui.FieldType.INLINEHTML ,
          label: 'Fulfillment'
        })

        const fileHTML = file.load({id: '/SuiteScripts/your-application/index.html'})

        contentHTML.defaultValue = fileHTML.getContents()

        context.response.writePage(form)
      }
    }

    return {
      onRequest: onRequest
    }
  })
```

# vuepload
Upload component for Vue.js 2.x

**NOTE : this was just a try with one of my projet, but I feel it could become a cool plugin. However, I started the implementation as a Vue component, not plugin, so I am sure yet of how to make the transition. As a consequence, installation remains manual (which sucks, I know). Any tip, recommandation or help is appreciated **

## Installation
Place Upload.js in your `./components` or whatever feels apropriate. In your `app.js` file, just load the component like so:
```javascript
Vue.component('vuepload', require('./components/Upload.vue'))
```

In `app.scss`, import Vuepload Sass file : `@import "resources/assets/sass/upload";`

## Dependencies

Axios is the default - and for now mandatory - HTTP client used here. It must be required globally like so :
``` javascript
window.axios = require('axios');
```

Bootstrap 3 CSS is also required for glyphicons.

## Use

### Minimum use

You can now use the component like any other Vue component, and you must at least provide HTML attribute *upload-id* with a unique id in the DOM, and a relative OR absolute upload url that accepts POST requests
```
<vuepload upload-id="photos" upload-url="/api/upload"></vuepload>
```

### Path
By default, Vuepload expects your API to respond with the newly created row in your DB corresponding to the uploaded file. This row should contain the path to the file in the field called "path". An example of a response accepted by Vuepload :
```
{
  id:1;
  path: 'images/64f6s654%fdf6sf8phvv$f.jpg'
}
```

If the file is actually stored in `public/storage/images`, as it can be with Laravel File system, you may provide a prefix like so :
```
<vuepload img-prefix="/storage" upload-id="photos" upload-url="/api/upload"></vuepload>
```


### Get file ids in upload callback
If you want to get back the DB ids of the files uploaded in an array, here is how you can do that :

```
<template>
  <vuepload upload-id="photos" upload-url="/api/upload" :file-ids="photo_ids" v-on:update:file-ids="updateFiles" ></vuepload>
</template>
<script>
module.exports = {
    data : function() {
            photo_ids : []
        }
    },
    methods : {
        updateFiles: function(val) {
            this.photo_ids = val;
        }
    }
}
```
The use of `updateFiles` function is explained in Vue.js doc [here](https://vuejs.org/v2/guide/components.html#sync-Modifier).

### Cover phto
If you want one of the photos to be chosen as "cover photo", just add HTML attribute
```
<vuepload cover="true" upload-id="photos" upload-url="/api/upload"></vuepload>
```

### Previously uploaded files
If upload is not only used to upload new files, but maybe to edit an existing element that is linked to various files that need management, it can be useful to make Vuepload display those previously uploaded files. In the same format accepted in the ***Path*** section, you may provide those with `old-files`.
```
<vuepload :old-files="oldFiles upload-id="photos" upload-url="/api/upload"></vuepload>
<script>
  module.exports = {
    created : function() {
      var vm = this;
      axios.get('/api/files').then(function(response) {
        vm.oldFiles = response.data;
      });
    },
    data : function() {
      return {
        oldFiles : []
      }
    }
  }
</script>
```

### Single / Multiple upload
By default, Vuepload enables you to upload multiple files. If you to upload only one file, add `single` attribute
```
<vuepload single="true" upload-id="photos" upload-url="/api/upload"></vuepload>
```




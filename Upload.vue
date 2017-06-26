<template>
    <div>
        <output :id="outputId" class="upload">
            <upload-thumb v-for="photo in files" :key="photo.id" :id="photo.id" :src="photo.path" :name="photo.name" :img-prefix="imgPrefix" v-on:remove-img="removeImg" v-on:star-img="starImg"></upload-thumb>
            <div :id="addId" class="upload add text-primary" v-on:click="triggerBrowse"><div class="upload thumb thumb-custom text-primary"><span class="glyphicon glyphicon-picture"></span></div></div>
        </output>
        <input v-if="single != 'true'" :id="uploadId" type="file" multiple v-on:change="filesAppended" style="display:none"></input>
        <input v-if="single == 'true'" :id="uploadId" type="file" v-on:change="filesAppended" style="display:none"></input>
    </div>
</template>

<script>

    module.exports = {
        props: ['uploadId', 'uploadUrl', 'fileIds', 'cover', 'imgPrefix', 'oldFiles', 'single'],
        mounted : function() {
            if (this.cover == 'true') {
                document.getElementById(this.outputId).className += ' with-cover';
            }
        },
        data : function () {
            return {
                files : []
            }
        },
        watch : {
            oldFiles : function() {
                this.files = this.oldFiles;
            }
        },
        computed : {
            outputId : function() {
                return this.uploadId + '-list';
            },
            addId : function() {
                return this.uploadId + '-add';
            }
        },
        methods : {
            triggerBrowse : function() {
                document.getElementById(this.uploadId).click();
            },
            filesAppended : function() {
                var files = document.getElementById(this.uploadId).files;

                if (this.single == 'true') {
                    this.files = [];
                    this.fileIds = [];
                }

                for (var i = 0, f; f = files[i]; i++) {
                    // Only process image files.
                    if (!f.type.match('image.*')) {
                        // continue;
                    }

                    this.upload(f, i, files.length);
                }
            },
            upload : function(file, i, length) {
                var vm = this;
                let data = new FormData();
                data.append('file', file);

                axios.post(this.uploadUrl, data).then(function (response) {
                    vm.files.push(response.data);
                    vm.fileIds.push(response.data.id);

                    if (i == length - 1) {
                        vm.$emit('update:file-ids', vm.fileIds);
                    }
                }).catch(function() {
                    vm.renderThumbError();
                });
            },
            renderThumbError : function() {
                var div = document.createElement('div');
                div.className = "thumb thumb-custom thumb-file thumb-error upload position-relative";
                div.innerHTML = '<span class="glyphicon glyphicon-remove"></span>'

                document.getElementById(this.outputId).insertBefore(div, null);
            },
            removeImg : function(id) {
                id = parseInt(id);
                for (var i in this.files) {
                    if (this.files[i].id == id) {
                        this.files.splice(i, 1);
                    }
                }
                this.fileIds.splice(this.fileIds.indexOf(id), 1);
                this.$emit('update:file-ids', this.fileIds);
            },
            starImg : function(id) {
                id = parseInt(id);
                for (var i in this.files) {
                    if (this.files[i].id == id) {
                        var newCover = this.files[i];
                        this.files.splice(i, 1);
                        this.files.unshift(newCover);
                    }
                }
                this.fileIds.splice(this.fileIds.indexOf(id), 1);
                this.fileIds.unshift(id);
                this.$emit('update:file-ids', this.fileIds);
            }
        }
    }

    Vue.component('upload-thumb', Vue.extend({
        template : [
            '<div :id="thumbId" class="upload thumb-container position-relative">',
            '<img class="thumb thumb-file upload" :src="dsrc" :title="name"/>',
            '<div class="upload thumb thumb-hover">',
            '<span :data-id="id" class="rm-upload glyphicon glyphicon-remove remove" v-on:click="removeImg"></span>',
            '<span :data-id="id" class="rm-upload glyphicon glyphicon-star star" v-on:click="starImg"></span>',
            '</div>',
            '</div>'
        ].join(''),
        props : ['id', 'src', 'name', 'imgPrefix'],
        computed : {
            thumbId : function() {
                return 'thumb-' + this.id;
            },
            dsrc : function() {
                var prefix = this.imgPrefix == undefined ? '' : this.imgPrefix;
                return prefix + '/' + this.src;
            }
        },
        methods : {
            removeImg : function(e) {
                this.$emit('remove-img', e.target.dataset.id);
            },
            starImg : function(e) {
                this.$emit('star-img', e.target.dataset.id);
            }
        }
    }));

</script>

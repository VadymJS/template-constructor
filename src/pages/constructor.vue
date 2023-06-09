<template>
    <div class="constructor">
        <Sidebar/>
        
        <div class="wrapper">
            <div class="constructor-body">
                <router-link to="/" class="btn btn-icon btn-link">
                    <img 
                        src="../assets/arrow-left.svg" 
                        class="svg-icon" 
                        alt="Arrow left" 
                        width="16" 
                        height="14"
                        @click="$router.push('/')">
                </router-link>
            
                <div v-if="template" class="constructor-body-content">
                    <div class="form-group">
                        <div class="custom-control">
                            <input
                                type="checkbox"
                                class="custom-control-input"
                                id="isVisible"
                                v-model="template.isVisible">
                            <label class="custom-control-label" for="isVisible">
                                <span v-if="template.isVisible">Visible</span>
                                <span v-else>Hidden</span>
                            </label>
                        </div>
                    </div>
                    <div class="constructor-body-upload form-group">
                        <UploadImages @changed="handleImages" ref="uploadImagesRef" />
                    </div>

                    <div class="form-row">
                        <div class="form-group" :class="{'is-invalid': validation.id !== null}">
                            <input 
                                v-model="template.id"
                                type="text" 
                                class="form-control" 
                                placeholder="###"
                                :readonly="editMode">
                            <div class="invalid-tooltip" v-if="validation.id !== null">
                                {{ validation.id }}
                            </div>
                        </div>

                        <div class="form-group" :class="{'is-invalid': validation.title !== null}">
                            <input 
                                v-model="template.title"
                                type="text" 
                                class="form-control" 
                                placeholder="Name">
                            <div class="invalid-tooltip" v-if="validation.title !== null">
                                {{ validation.title }}
                            </div>
                        </div>
                    </div>

                    <div class="form-group" :class="{'is-invalid': validation.url !== null}">
                        <input
                            v-model="template.url"
                            type="text" 
                            class="form-control" 
                            placeholder="https://design###.horoshop.ua/">
                        <div class="invalid-tooltip" v-if="validation.url !== null">
                            {{ validation.url }}
                        </div>
                    </div>
                </div>
            </div>

            <div v-if="editMode" class="constructor-buttons">
                <button
                    @click="deleteTemplate"
                    class="btn btn-bordered">
                    Delete
                </button>
                <button 
                    @click="editTemplate"
                    class="btn btn-info">
                    Save and exit
                </button>
            </div>
            <div v-else class="constructor-buttons">
                <button 
                    @click="addTemplate"
                    class="btn btn-info">
                    Save and exit
                </button>
            </div>
        </div>
    </div>
</template>

<script setup>

import { useRoute, useRouter } from 'vue-router'
import { ref, onMounted, computed } from 'vue';
import Sidebar from '../components/Sidebar.vue';
import { useStore } from "vuex";
import UploadImages from "vue-upload-drop-images";

const route = useRoute();
const router = useRouter();
const store = useStore();

const editMode = ref(false);
const template = ref({});
const templateCopy = computed(() => {
    if (route.params?.id) {
        return store.getters.getTemplateById(route.params.id)
    } else {
        return {
            isVisible: true,
        };
    }
});
const validation = ref({
    id: null,
    title: null,
    url: null,
})

function addTemplate() {
    validation.value.id = null;
    validation.value.title = null;
    validation.value.url = null;

    if (!template.value.id || !template.value.title || !template.value.url) {
        if (!template.value.id) {
            validation.value.id = 'ID cant be empty';
        }
        if (!template.value.title) {
            validation.value.title = 'Title cant be empty';
        }
        if (!template.value.url) {
            validation.value.url = 'Url cant be empty';
        }
        return;
    }

    if (isNaN(+template.value.id)) {
        validation.value.id = 'Use digits for ID';
        return;
    }

    store.state.templates.forEach((template) => {
        if (parseFloat(templateCopy.value.id) === parseFloat(template.id)) {
            validation.value.id = 'Already had this ID';
            return;
        }
    })

    if (!validation.value.id) {
        store.dispatch('addTemplate', template.value);
        router.push('/');
    }
}

function editTemplate() {
    store.dispatch('editTemplate', {oldObj: templateCopy.value, copyObj: template.value});
    router.push('/');
}

function deleteTemplate() {
    store.dispatch('deleteTemplate', templateCopy.value.id);
    router.push('/');
}

const uploadImagesRef = ref(null);

function handleImages(files) {
    setTimeout(() => {
        template.value.imgUrl = uploadImagesRef.value.Imgs;
    }, 500);
}

onMounted(() => {
    template.value = templateCopy.value;

    if (route.params?.id) {
        if (store.state.templates.length) {
            editMode.value = true;

            setTimeout(() => {
                if (template.value.imgUrl?.length) {
                    uploadImagesRef.value.Imgs = template.value.imgUrl; 
                }
            }, 500)
        } else {
            router.push('/');
        }
    } 
})

</script>
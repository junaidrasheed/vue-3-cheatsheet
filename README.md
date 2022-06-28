# vue-3-cheatsheet

# Cheat Sheet for Vue 3 Composition API

This guide contains the cheat sheet for transitioning from Options API to Compistion API. The guide will be solely focused on `<script setup>` syntax.


## Defining Data
>**ProTip:** Options API does not allow to create non-reactive data, but Composition API does.
#### Options API
```
data() {
	return {
		count: 0,
		messages: []
	}
}
```
#### Composition API
Use `ref()` for primitive data types and `reactive()` for arrays and objects. 
```
import {ref, reactive } from 'vue'
const count = ref(0);
const messages = reactive([])
```

## Defining Props
#### Options API
```
props:{
	count: { type: Number, default: 0 },
	messages: { type: Array, default: () => [] }
}
```
#### Composition API
> **defineProps()** is available by default, so no need to import
```
const props = defineProps({
	count: { type: Number, default: 0 },
	messages: { type: Array, default: () => [] }
})
```

## Defining Computed and Watch
#### Options API
```
computed: {
	unreadCount(){
		//messages is a data property
		return this.messages.unread;
	}
},
watch: {
	unreadCount(newVal, oldVal){
		this.sendNotification(newVal)
	}
}
```
#### Composition API
```
import { watch, computed } from 'vue'
const unreadCount = computed(() => {
	//messages is a reactive() variable
	return messages.unread
})
watch(unreadCount, (newVal, oldVal) => {
	sendNotification(newVal)
})
```

## Defining methods
#### Options API
```
methods: {
	sendNotification(unreadCount){
		//send user notification about unread messages
	}
}
```
#### Composition API
```
function sendNotification(unreadCount){
	//send user notification about unread messages
}
```

## Sending Events from Child to Parent
#### Options API
|Template        |Script                         |
|----------------|-------------------------------|
|`$emit('custom-event', payload)`|`this.$emit('custom-event',payload)`            |

#### Composition API
First in script section define all your emits for the component.
```
const emit = defineEmits(['custom-event'])
```
Then you can simple use `emit('custom-event',payload)` from both `<template>` and `<script>` sections.

> In case of global events, Vue 2 allows us to create event bus like `export const bus = new Vue();` but in Vue 3 as suggested by [official docs](https://v3-migration.vuejs.org/breaking-changes/events-api.html#event-bus) we can use `mitt` library. Which we can then add as a composeable API to our components.

First create a composeable API file,  in this case `composeables/useEmitter.js`
```
import { getCurrentInstance } from 'vue'
export default function useEmitter() {
    const internalInstance = getCurrentInstance(); 
    const emitter = internalInstance.appContext.config.globalProperties.emitter;
    return emitter;
}
```
And then in our components
```
import useEmitter from '@/composables/useEmitter'
const emitter = useEmitter()

//listening to event
emitter.on('custom-event', () => {})

//emitting event
emitted.emit('custom-event', payload)
```


## Provide / Inject
Common practice to pass down data from parent to their nested child components
#### Options API
Not supported out of the box, but can be added as an add-on
```
//from parent, where message is a data property
//following example is for passing non-reactive data
data(){
	return {
		message: 'Hello!'
	}
},
provide(){
	return {
		'message': this.message			//non-reactive,
		'message': computed(() => this.message)		//reactive
	}
}
//from child
inject: ['message']
```
#### Composition API
```
//from parent
import { ref, provide } from 'vue'

const message = ref('Hello!')
provide('message', message)

//from child
import { inject } from 'vue'
const message = inject('message')
```

## Using store (Vuex)
#### Options API
Options API give us access to $store variable that can be used in `<template>` as well as `<script>`
```
this.$store.dispatch('someMethod')
```
#### Composition API
In Composition API we don't have access to this. Instead we use `useStore` API
```
import { useStore } from 'vue';
const store = useStore();
store.dispatch('someMethod');
```

## Using router
#### Options API
Options API give us access to $router & $route variable that can be used in `<template>` as well as `<script>`
```
this.$router.push('/url');
this.$route.params.id;
```
#### Composition API
In Composition API we don't have access to this. Instead we use `useRoute` and `useRouter` APIs
```
import { useRouter, useRoute } from 'vue'
const route = useRoute();
const router = useRouter();

const parameter_id = route.params.id;
router.push('/url');
```


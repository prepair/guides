# Component Guide for Vue "1.5"

- [Component Guide for Vue "1.5"](#component-guide-for-vue-15)
  - [Do's and Don'ts](#dos-and-donts)
    - [Don't use `.sync` on `v-bind` option](#dont-use-sync-on-v-bind-option)
    - [Don't use `twoWay` prop option](#dont-use-twoway-prop-option)
    - [Don't mutate props in **vm**](#dont-mutate-props-in-vm)
    - [Don't mutate props via `v-model` in template](#dont-mutate-props-via-v-model-in-template)
    - [Don't mutate props via **expression** in template](#dont-mutate-props-via-expression-in-template)
    - [Don't mutate elements of **array prop** or the array itself](#dont-mutate-elements-of-array-prop-or-the-array-itself)
    - [Don't mutate properties of an **object prop**](#dont-mutate-properties-of-an-object-prop)
    - [Don't mutate **store data** directly](#dont-mutate-store-data-directly)
    - [Don't use `this.$dispatch`](#dont-use-thisdispatch)
    - [Don't use `this.$broadcast`](#dont-use-thisbroadcast)
    - [Don't use `events` option](#dont-use-events-option)
    - [Use filters only in mustache interpolation and `v-bind` expressions](#use-filters-only-in-mustache-interpolation-and-v-bind-expressions)
    - [Don't use interpolation within attributes](#dont-use-interpolation-within-attributes)
    - [Use `v-html` instead of HTML interpolation](#use-v-html-instead-of-html-interpolation)
  - [Tools](#tools)
    - [$emit](#emit)
    - [emit/emitOnNextTick/eventBusMixin](#emitemitonnexttickeventbusmixin)
    - [get/set computed](#getset-computed)
    - [map-one-way-prop/map-one-way-props](#map-one-way-propmap-one-way-props)
    - [v-model](#v-model)
    - [~~v-one-way-model~~](#v-one-way-model)

## Do's and Don'ts

### Don't use `.sync` on `v-bind` option

- Apply one way data flow. See [Don't mutate props in vm](#dont-mutate-props-in-vm).

### Don't use `twoWay` prop option

- Apply one way data flow. See [Don't mutate props in vm](#dont-mutate-props-in-vm).

### Don't mutate props in **vm**

```javascript
// NOK
props: {
  externalProp: String
},

methods: {
  someMethod () {
    // don't mutate props!
    this.externalProp = 'newValue'
  }
}
```

```javascript
// OK
// $emit
methods: {
  someMethod () {
    this.$emit('update:external-prop', 'newValue')
  }
}
```

```javascript
// OK
// $emit through update method
methods: {
  someMethod () {
    this.updateExternalProp('newValue')
  },

  updateExternalProp (value) {
    this.$emit('update:external-prop', value)
  }
}
```

```javascript
// OK
// mapOneWayProp creator
computed: {
  // will emit 'update:external-prop' event by default with the new value
  externalPropInt: mapOneWayProp('externalProp')
},

methods: {
  someMethod () {
    // you can assign safely
    this.externalPropInt = 'newValue'
  },
}
```

```javascript
// OK
// mapOneWayProps creator
computed: {
  ...mapOneWayProps({
    'externalProp1Int': 'externalProp1', // will emit 'update:external-prop1' event by default with the new value
    'externalProp2Int': 'externalProp2', // will emit 'update:external-prop2' event by default with the new value
  })
},

methods: {
  someMethod () {
    // you can assign safely
    this.externalProp1Int = 'newValue1'
    this.externalProp2Int = 'newValue2'
  },
}
```

```javascript
// OK
// get/set computed
computed: {
  externalPropInt: {
    get () {
      return this.externalProp
    },

    set (value) {
      this.$emit('update:external-prop', value)
      // or
      // this.updateExternalProp(value)
    }
  }
},

methods: {
  someMethod () {
    // you can assign safely
    this.externalPropInt = 'newValue'
  },

  // updateExternalProp (value) { ... }
}
```

```javascript
// OK
// local data (initialized once, parent notified)
data () {
  return {
    someProp: this.externalProp // set initial value
  }
},

methods: {
  someMethod () {
    // you can assign safely
    this.someProp = 'newValue'
    // if you want to update the externalProp too
    this.$emit('update:external-prop', value)
    // or
    // this.updateExternalProp(value)
  },

  // updateExternalProp (value) { ... }
}
```

### Don't mutate props via `v-model` in template

```javascript
// NOK

// component/index.js
props: {
  externalProp: String
}

// component/template.html
// dont use v-model directly with props
<input type="text" v-model="externalProp">
```

<strike>

```javascript
// OK
// v-one-way-model

// component/index.js
props: {
  externalProp: String
}

// component/template.html
<input type="text" v-one-way-model="externalProp"> <!-- will emit `update:external-prop` event by default -->
```

</strike>

```javascript
// OK
// one way computed + v-model

// component/index.js
props: {
  externalProp: String
},

computed: {
  externalPropInt: mapOneWayProp('externalPropInt')
}

// component/template.html
<input type="text" v-model="externalPropInt">
```

- You may use additional techniques from [Don't mutate props in **vm**](#dont-mutate-props-in-vm)

### Don't mutate props via **expression** in template

```javascript
// NOK

// component/index.js
props: {
  externalProp: String
}

// component/template.html
// don't assign values directly to props
<button @click="externalProp='newValue'">
```

```javascript
// OK
// handler method

// component/index.js
props: {
  externalProp: String
},

methods: {
  handleClick () {
    this.$emit('update:external-prop', 'newValue')
  }
}

// component/template.html
<button @click="handleClick">
```

- You may use other techniques from [Don't mutate props in **vm**](#dont-mutate-props-in-vm)

### Don't mutate elements of **array prop** or the array itself

```javascript
// NOK
// foo/index.js
props: {
  externalArrayProp1: Array
},

methods: {
  someMethods () {
    // don't mutate array prop elements
    this.externalArrayProp1[0] = 'newValue1'

    // don't mutate array prop object element properties
    this.externalArrayProp1[0].prop1 = 'newValue2'

    // don't call mutable operations on array prop
    this.externalArrayProp1.push(...)
    this.externalArrayProp1.sort(...)
    this.externalArrayProp1.splice(...)
    // ...
  }
},

// foo/template.html
// don't mutate array prop elements
<input type="text" v-model="externalArrayProp1[0]"></input>

// don't mutate array prop object element properties
<input type="text" v-model="externalArrayProp1[0].prop1"></input>

// don't call mutable operations on array prop
<button @click="externalArrayProp1.sort()"><button>
```

- To avoid these use techniques from [Don't mutate props in **vm**](#dont-mutate-props-in-vm)
  - or [Don't mutate props via v-model in template](#dont-mutate-props-via-v-model-in-template)
  - or [Don't mutate props via expression in template](#dont-mutate-props-via-expression-in-template)

### Don't mutate properties of an **object prop**

```javascript
// NOK
// foo/index.html
props: {
  externalObjectProp1: Object
},

methods: {
  someMethods () {
    // don't mutate properties of an object prop
    this.externalObjectProp1.prop1 = 'newValue1'

    // don't add new properties to an object prop
    this.externalObjectProp1.newProp1 = 'newValue2'

    // don't remove properties from an object prop
    delete this.externalObjectProp1.prop2
  }
}

// foo/template.html
// don't mutate properties of an object prop
<input type="text" v-model="externalObjectProp1.prop1"></input>
```

- To avoid these use techniques from [Don't mutate props in **vm**](#dont-mutate-props-in-vm)
  - or [Don't mutate props via v-model in template](#dont-mutate-props-via-v-model-in-template)
  - or [Don't mutate props via expression in template](#dont-mutate-props-via-expression-in-template)

### Don't mutate **store data** directly

```javascript
// NOK
vuex: {
  getters: {
    vuexProp1: state => state.some.prop1
  }
},

methods: {
  someMethod () {
    // don't mutate store data directly
    this.vuexProp1 = 'newValue1'
    // nor do this
    this.$store.state.some.prop2 = 'newValue2';
  }
}

// also don't mutate them with `v-model` or expressions in template
```

- Use Vuex actions or mutations depending on your scenario. Here `someMethod` could be transformed into an action or mutation for example

- Mutating via `storeLinker` is a special exception (for now) but still, try to use Vuex instead if possible (mainly in new components)

- You may use and adapt techniques from [Don't mutate props in **vm**](#dont-mutate-props-in-vm)
  - or [Don't mutate props via v-model in template](#dont-mutate-props-via-v-model-in-template)
  - or [Don't mutate props via expression in template](#dont-mutate-props-via-expression-in-template)

### Don't use `this.$dispatch`

- Use `$emit`(s) if you want to reach some of the parent components. You may need to re-emit the same or another event per parent.
- Use `emit`/`emitOnNextTick` if you want to reach a component/thing outside of the actual component hierarchy
- Use Vuex action (action/mutation)

### Don't use `this.$broadcast`

- Use `emit`/`emitOnNextTick`
- Use Vuex action (action/mutation)

### Don't use `events` option

- Use `v-on` (`@`) in the template if the event emitted with `$emit`. (You may use `$on` in vm in some cases but `@` is preferred)
- Use `eventBusMixin` if the event emitted with `emit`/`emitOnNextTick`. (You may use `on` as well but you have to clean up after yourself which is done by `eventBusMixin` automatically so that is the preferred way)
- Use **Vuex** getters to react changes made by **Vuex** action if that is the case

### Use filters only in mustache interpolation and `v-bind` expressions

```html
<!-- in mustaches -->
{{ message | capitalize }}

<!-- in v-bind -->
<div :id="rawId | formatId"></div>
```

### Don't use interpolation within attributes

```html
<!-- NOK -->
<button class="btn btn-{{ size }}"></button>

<!-- OK -->
<button :class="'btn btn-' + size"></button>

<!-- OK -->
<button :class="buttonClasses"></button>

computed: {
  buttonClasses () {
    return 'btn btn-' + size
  }
}
```

### Use `v-html` instead of HTML interpolation

- so replace `{{{ somePropWithHtml }}}` with `v-html`

## Tools

### $emit

- Useful to notify parent component about something (update of **props**, other events, etc.)

**Doc**: [API/vm.$emit](https://v1.vuejs.org/api/#vm-emit)

### emit/emitOnNextTick/eventBusMixin

- Useful for emitting global events
- Useful to notify components outside of the current component hierarchy (parents should be notified via `$emit`)
- To listen on events in components prefer using `eventBusMixin` instead of `on`, because it cleans up automatically on component destroy
- **Note**: the events are emitted outside of the component vue life cycle which means that some component prop may not changed when the listener runs, in this cases `emitOnNextTick` may help

**Doc**: `app/scripts/util/event-bus`

**Doc**: `app/scripts/mixins/event-bus`

### get/set computed

```javascript
computed: {
  someProp: {
    get () {
      // return local data
      // return other computed
      // return vuex prop
      // ...
    },

    set (value) {
      // set local data
      // $emit event
      // call vuex action
      // emit global event
      // ...
    }
  }
}
```

- **Doc**: [Guide/Computed Setter](https://v1.vuejs.org/guide/computed.html#Computed-Setter)

**Pros**

- universal power tool
- plays well with `v-model` and/or **prop** mutation in **vm**

**Cons**

- you need to create a **new prop** with a **new name** (`isLoading` => `isLoadingInt`)
- could be boilerplate => use the more compact [`mapOneWayProp`/`mapOneWayProps`](#map-one-way-propmap-one-way-props) if applicable

### map-one-way-prop/map-one-way-props

**Doc**: `app/scripts/utils/component/map-one-way-prop`

**Doc**: `app/scripts/utils/component/map-one-way-props`

**Pros**

- plays well with `v-model` and/or **prop** mutation in **vm**
- compact

**Cons**

- you need to create a **new prop** with a **new name** (`isLoading` => `isLoadingInt`)
- could be to simple, if you need more power => [**get/set computed**](#getset-computed)

### v-model

- Watch out because it is a **two way** directive so it modifies the bound prop (which is ok for **local data** but NOK for **props**)
- It can be used with **one way** data flow if the **prop** is a **get/set computed**
- It can be used freely with **local data**

**Doc**: [API/v-model](https://v1.vuejs.org/api/#v-model)

### ~~v-one-way-model~~

**Note**: use [**get/set computed**](#getset-computed) or [**mapOneWayProp**/**mapOneWayProps**](#map-one-way-propmap-one-way-props) instead until this gets implemented properly in Vue 2 (vue2 fork)

<strike>

**Doc**: `app/scripts/directives/one-way-model`

**Pros**

- works the same way as `v-model` except it is a **one way** directive so it emits an **event** and/or calls a **callback** method on change
- you don't need to create a **new prop** and think of a **new name** for it

**Cons**

- it only works well with templates, if you want to mutate a **prop** from the **vm** or from the **vm** too you may use [**get/set computed**](#getset-computed) or [`mapOneWayProp`/`mapOneWayProps`](#map-one-way-propmap-one-way-props)

</strike>

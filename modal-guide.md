# Modal Guide for Vue "1.5"

- [Modal Guide for Vue "1.5"](#modal-guide-for-vue-15)
  - [Types of modals](#types-of-modals)
    - [base modal](#base-modal)
    - [confirmation modal](#confirmation-modal)
    - [article modal](#article-modal)
  - [Showing/hiding modals](#showinghiding-modals)
    - [Modal is always present in the DOM (no `v-if`)](#modal-is-always-present-in-the-dom-no-v-if)
      - [Using **local data**](#using-local-data)
      - [Using **global events**](#using-global-events)
      - [Using **store**](#using-store)
    - [Modal is only in the DOM when it needs to be displayed (`v-if`)](#modal-is-only-in-the-dom-when-it-needs-to-be-displayed-v-if)
      - [Using **local data**](#using-local-data)
      - [Using **global events**](#using-global-events)
      - [Using **store**](#using-store)
  - [Passing **props** to modals](#passing-props-to-modals)
    - [read only **props**](#read-only-props)
      - [Via **props**](#via-props)
      - [Via **global events**](#via-global-events)
      - [Via **store**](#via-store)
    - [**props** that will change](#props-that-will-change)
      - [Via **props**](#via-props)
      - [Via **global events**](#via-global-events)
      - [Via **store**](#via-store)

## Types of modals

### base modal

- Has two modes based on if you provide a `title` attribute or not
- If you provide the `title` attribute you will get a nice closable header above your content, if not your content will be displayed as it is.

**Doc**: `app/scripts/components/modal`

### confirmation modal

- based on `modal`
- `isVisible`, `isVisibleStorePath` and `isLoading` props are passed to `modal`

**Src**: `app/scripts/components/confirm`

### article modal

- based on `modal`
- `isVisible`, `isVisibleStorePath` and `isLoading` props are passed to `modal`

**Src**: `app/scripts/components/article-modal`

## Showing/hiding modals

### Modal is always present in the DOM (no `v-if`)

**Pros**

- ?

**Cons**

- Modal is always "alive" which could be a bad thing (things may start to work when not needed, react to global events, store changes, prop changes, ...)

#### Using **local data**

```javascript
// foo/index.js
data () {
  return {
    isBarModalVisible: false
  }
},

methods () {
  showBarModal () {
    this.isBarModalVisible = true
  },

  // optional, if you want to hide the modal explicitly
  hideBarModal () {
    this.isBarModalVisible = false
  },

  updateIsBarModalVisible (value) {
    this.isBarModalVisible = value
  }
}

// foo/template.html
<bar-modal
  :is-visible="isBarModalVisible"
  @update:is-visible="updateIsBarModalVisible"
></bar-modal>
```

```javascript
// bar-modal/index.js
props: {
  isVisible: {
    type: Boolean,
    required: true
  }
},

methods: {
  updateIsVisible (value) {
    this.$emit('update:is-visible', value)
  }
}

// bar-modal/template.html
<modal
  :is-visible="isVisible"
  @update:is-visible="updateIsVisible"
  // ...
>
  // ...
<modal>
```

**Pros**

- No store/no global events needed

**Cons**

- Bit verbose
- You can only open the modal from the containing component (which could be a perfectly valid use case though)

#### Using **global events**

```javascript
// foo/index.js
methods () {
  showBarModal () {
    emit('showBarModal')
  },

  // optional, if you want to hide the modal explicitly
  hideBarModal () {
    emit('hideBarModal')
  }
}

// foo/template.html
<bar-modal></bar-modal>
```

```javascript
// bar-modal/index.js
mixins: [
  eventBusMixin({
    showBarModal: 'show',
    hideBarModal: 'hide', // optional, to explicitly hide the modal from outside
  })],

data () {
  return {
    isVisible: false
  }
},

methods: {
  show () {
    this.isVisible = true
  },

  hide () {
    this.isVisible = false
  },

  updateIsVisible (value) {
    this.isVisible = value
  }
}

// bar-modal/template.html
<modal
  :is-visible="isVisible"
  @update:is-visible="updateIsVisible"
  // ...
>
  // ...
<modal>
```

**Pros**

- You can open the same modal from many places without much hassle

**Cons**

- Modal component should be present in the DOM when the event goes out

#### Using **store**

```javascript
// foo/index.js
// trigger volatile.isBarModalVisible state change somehow
//  - via Vuex action/mutation
//  - via store linker

// foo/template.html
<bar-modal></bar-modal>
```

```javascript
// bar-modal/index.js
// yes, no extra code needed

// bar-modal/template.html
<modal
  is-visible-store-path="volatile.isBarModalVisible"
  // ...
>
  // ...
<modal>
```

**Pros**

- Shortest, minimum effort solution
- You can open the same modal from many places without much hassle

**Cons**

- Modal component should be present in the DOM when an action/mutation changes the visibility flag

### Modal is only in the DOM when it needs to be displayed (`v-if`)

**Note** Transitions may not work with this approach.

**Pros**

- Modal is not "alive" when it is not visible so it won't react to global events, store changes, prop changes, etc.

**Cons**

- ?

#### Using **local data**

```javascript
// foo/index.js
data () {
  return {
    isBarModalVisible: false
  }
},

methods () {
  showBarModal () {
    this.isBarModalVisible = true
  },

  // optional, if you want to hide the modal explicitly
  hideBarModal () {
    this.isBarModalVisible = false
  },

  updateIsBarModalVisible (value) {
    this.isBarModalVisible = value
  }
}

// foo/template.html
<bar-modal
  v-if="isBarModalVisible"
  :is-visible="isBarModalVisible"
  @update:is-visible="updateIsBarModalVisible"
></bar-modal>
```

```javascript
// bar-modal/index.js
props: {
  isVisible: {
    type: Boolean,
    required: true
  }
},

methods: {
  updateIsVisible (value) {
    this.$emit('update:is-visible', value)
  }
}

// bar-modal/template.html
<modal
  :is-visible="isVisible"
  @update:is-visible="updateIsVisible"
  // ...
>
  // ...
<modal>
```

**Pros**

- No store/no global events needed
- Single visibility flag can be used

**Cons**

- Bit verbose
- You can only open the modal from the containing component (which could be a perfectly valid use case though)

#### Using **global events**

```javascript
// foo/index.js
data () {
  return {
    isBarModalVisible: false
  }
},

methods () {
  showBarModal () {
    this.isBarModalVisible = true
    // we need to wait for the modal to be created
    emitOnNextTick('showBarModal')
  },

  // optional, if you want to hide the modal explicitly
  hideBarModal () {
    this.isBarModalVisible = false
  }
}

// foo/template.html
<bar-modal v-if="isBarModalVisible"></bar-modal>
```

```javascript
// bar-modal/index.js
mixins: [
  eventBusMixin({
    showBarModal: 'show',
    hideBarModal: 'hide', // optional, to explicitly hide the modal from outside
  })],

data () {
  return {
    isVisible: false
  }
},

methods: {
  show () {
    this.isVisible = true
  },

  hide () {
    this.isVisible = false
  },

  updateIsVisible (value) {
    this.isVisible = value
  }
}

// bar-modal/template.html
<modal
  :is-visible="isVisible"
  @update:is-visible="updateIsVisible"
  // ...
>
  // ...
<modal>
```

**Pros**

- ?

**Cons**

- You need an extra visibility flag
- You need some extra effort to be able to open the same modal from many places

#### Using **store**

```javascript
// foo/index.js
// wire-in volatile.isBarModalVisible as prop somehow
//  - via Vuex getter
//  - via store linker
//
// trigger volatile.isBarModalVisible state change somehow
//  - via Vuex action/mutation
//  - via store linker

// foo/template.html
<bar-modal v-if="isBarModalVisible"></bar-modal>
```

```javascript
// bar-modal/index.js
// yes, no extra code needed

// bar-modal/template.html
<modal
  is-visible-store-path="volatile.isBarModalVisible"
  // ...
>
  // ...
<modal>
```

**Pros**

- Shortest, minimum effort solution
- You can open the same modal from many places without much hassle

**Cons**

- ?

## Passing **props** to modals

### read only **props**

#### Via **props**

```html
<bar-modal :prop1="..." :prop2="..."></bar-modal>
```

**Pros**

- ?

**Cons**

- Will bind these props permanently so if they change it may cause some wanted/unwanted change inside the modal (which could be a problem when used with modals that always present in the DOM)

#### Via **global events**

```javascript
// foo/index.js
methods: {
  showBarModal () {
    emit('showBarModal', {
      param1: 'someValue1',
      param2: 'someValue2'
    })
  }
}

// bar-modal/index.js
methods: {
  mixins: [
    eventBusMixin({ showBarModal: 'show' })
  ],

  data () {
    return {
      someProp1: '',
      someProp2: ''
    }
  },

  methods: {
    show (payload = {}) {
      let { param1, param2 } = payload
      // store the parameters as local data or do whatever you want with them
      this.someProp1 = param1
      this.someProp2 = param2
    }
  }
}
```

**Pros**

- Flexible, parameters are not bound

**Cons**

- Manual parameter processing is a bit overhead

#### Via **store**

```javascript
// bar-modal/index.js

// wire in desired store state via
//  - Vuex getters
//  - store linker
```

**Pros**

- Easy/peasy without extra overhead
- Flexible, parameters are not bound

**Cons**

- ?

### **props** that will change

#### Via **props**

```javascript
// foo/template.html
<bar-modal
  :prop1="..."
  :prop2="..."
  @update:prop1="..."
  @update:prop1="..."
></bar-modal>

// bar-modal/index.js
methods: {
  someMethod () {
    this.$emit('update:prop1', 'newValue1')
    this.$emit('update:prop2', 'newValue2')
  }
}
```

#### Via **global events**

```javascript
// foo/index.js
methods: {
  showBarModal () {
    emit(
      'showBarModal',
      {
        param1: 'someValue1',
        param2: 'someValue2',
        onParam1Update: () => { ... },
        onParam2Update: () => { ... }
      }
    )
  }
}

// bar-modal/index.js
methods: {
  mixins: [
    eventBusMixin({ showBarModal: 'show' })
  ],

  data () {
    return {
      someProp1: '',
      someProp2: '',
      onParam1Update: () => {},
      onParam2Update: () => {},
    }
  },

  methods: {
    show (payload = {}) {
      let { param1, param2, onParam1Update, onParam2Update } = payload
      // store the parameters as local data or do whatever you want them
      this.someProp1 = param1
      this.someProp2 = param2
      this.onParam1Update = onParam1Update
      this.onParam2Update = onParam2Update
    },

    someMethod () {
      this.onParam1Update('newValue1')
      this.onParam2Update('newValue2')
    }
  }
}
```

**Pros**

- ?

**Cons**

- Manual parameter processing is a bit overhead

#### Via **store**

```javascript
// bar-modal/index.js

// wire in desired store state via
//  - Vuex getters
//  - store linker
//
// trigger state change somehow
//  - via Vuex action/mutation
//  - via store linker
```

**Pros**

- Easy/peasy without extra overhead

**Cons**

- ?

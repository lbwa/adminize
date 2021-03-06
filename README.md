<h1 align="center">Vue auth boilerplate</h1>

<p align="center">
  <a href="https://github.com/lbwa/vue-auth-boilerplate/actions">
    <img alt="github actions - tests" src="https://github.com/lbwa/vue-auth-boilerplate/workflows/Tests/badge.svg">
  </a>
  <a href="https://lbwa.github.io/vue-auth-boilerplate">
    <img alt="github actions - deployment" src="https://github.com/lbwa/vue-auth-boilerplate/workflows/Deployment/badge.svg">
  </a>
  <a href="https://nodejs.org/en/download/">
    <img alt="nodejs version" src="https://img.shields.io/badge/Node.js%20Version-%3E=8.9.0-ff69b4?logo=node.js&labelColor=3a424a"/>
  </a>
</p>

`Vue.js` console boilerplate with authentication.

<!-- TOC -->

- [Prerequisites](#prerequisites)
- [Development](#development)
- [Production build](#production-build)
  - [CDN supporting](#cdn-supporting)
- [User authentication](#user-authentication)
- [Symbolic constants](#symbolic-constants)
- [Environment variables](#environment-variables)
- [Declarations](#declarations)
- [Layouts](#layouts)
  - [Use route-based layout](#use-route-based-layout)
  - [Use non-route-based layout](#use-non-route-based-layout)
- [Components](#components)
- [Plugins](#plugins)
- [Effects](#effects)
- [Router](#router)
  - [Route config](#route-config)
- [State management](#state-management)
  - [Automatic registration](#automatic-registration)
  - [With request effects](#with-request-effects)
  - [Handle error](#handle-error)
  - [History store module](#history-store-module)
- [Changelog](#changelog)
- [License](#license)

<!-- /TOC -->

## Prerequisites

Please make sure you have installed [node.js](https://nodejs.org) version _8.9_ or above (the LTS version recommended).

## Development

- install all dependencies

  ```bash
  $ npm i
  ```

- start development server

  ```bash
  $ npm run serve
  ```

  Frontend server is running at `http://localhost:8080` and `http://<YOUR_DEVICE_LOCAL_IP>:8080` with [hot module replacement](https://webpack.js.org/concepts/hot-module-replacement/).

- run unit tests

  ```bash
  $ npm run test:unit
  ```

## Production build

```bash
$ npm run build
# yarn build
```

### CDN supporting

This project production build has supported third-party CDN libraries out of the box. All JS/CSS library CDN urls should be recorded by [third-parties.js](./third-parties.js). syntax like:

```ts
interface CDNUrl {
  name: string // required for js, npm package name
  library: string // required for js, global variable name in the browser
  js: string // js cdn file urls
  css: string // css cdn file urls
}
```

- JS libraries

  ```diff
  const thirdParties = [
    // ... other js/css libraries
  +  {
  +    name: 'vue',
  +    library: 'Vue',
  +    js: 'https://cdn.jsdelivr.net/npm/vue@2.6.x/dist/vue.min.js'
  +  }
  ]
  ```

- Pure CSS libraries

  step 1. You should import pure css libraries with tree-shaking.

  ```diff
  - import 'normalize.css'
  + if (__DEV__) { // __DEV__ is an environment variable
  +    require('normalize.css')
  + }
  ```

  step 2. add css library CDN link into [third-parties.js](./third-parties.js).

  ```diff
  const thirdParties = [
    // ... other js/css libraries
  +  {
  +    css: 'https://cdn.jsdelivr.net/npm/normalize.css@8.0.1/normalize.min.css'
  +  }
  ]
  ```

## User authentication

Please refer to `v-access` [documentation](https://github.com/lbwa/v-access#readme).

## Symbolic constants

A symbolic constant is a name given to a constant literal value. It's usually used to prevent [magic numbers][wiki-magic-number] and [hard-coding][wiki-hard-coding]. All symbolic constants should be recorded in the [src/constants.ts](src/constants.ts) file.

- Q: When should I use symbolic constants?

- A: [Feature flag](https://en.wikipedia.org/wiki/Feature_toggle), public settings, etc.

[wiki-magic-number]: https://en.wikipedia.org/wiki/Magic_number_(programming)
[wiki-hard-coding]: https://en.wikipedia.org/wiki/Hard_coding

## Environment variables

All `node.js` use-defined environment should be record at the `<PROJECT_ROOT>/.env*` files and start with `VUE_APP_*` prefix character. You can find more details from the [@vue/cli](https://cli.vuejs.org/guide/mode-and-env.html) documentation.

## Declarations

All global `TypeScript` declarations should be recorded in the [src/global.d.ts](src/global.d.ts) file.

This boilerplate has supported multiple declarations for CSS pre-process languages in [src/global.d.ts](src/global.d.ts).

## Layouts

In practice, we perhaps have two different kinds of layout components. All layout components should be placed in the [src/layouts](src/layouts) directory.

|           type           |       description        |      character       |
| :----------------------: | :----------------------: | :------------------: |
|   `route-based` layout   |  with `<router-view/>`   | `src/layouts/R*.vue` |
| `non-route-based` layout | without `<router-view/>` | `src/layouts/L*.vue` |

### Use route-based layout

A valid router config should be based on the following structure:

```ts
interface ConsoleRouteConfig extends RouteConfig {
  meta?: Partial<{
    layout: string // which route-based layout need to be rendered
    hidden: boolean
    icon: string
    title: string
  }>
}
```

We use `meta.layout` to decide which layout component we need to be rendered.

### Use non-route-based layout

As opposed to `route-based` layout, you should always import `non-route-based` layout component manually.

```ts
import { NonRouteBasedLayout } from '@/layouts/NonRouteBasedLayout'

export default {
  name: 'AnyViewComponent',

  components: {
    NonRouteBasedLayout
  }
}
```

## Components

We also have multiple kinds of components and should be placed in [src/components](src/components) directory.

| component type |                    description                    |              example              |      character       |
| :------------: | :-----------------------------------------------: | :-------------------------------: | :------------------: |
| presentational | represent user interface view excluding any state |          basic component          |     `Base*.vue`      |
|   container    |        state including all business logic         | any sub `views/layouts` component | `[VIEW_PREFIX]*.vue` |

## Plugins

If any `Vue.js` [plugin][doc-vue-plugin] exists, it should be placed in [src/plugins](src/plugins) directory.

[doc-vue-plugin]: https://vuejs.org/v2/guide/plugins.html

## Effects

All HTTP request function should be placed in [src/effects](src/effects) directory, and should be occurred by [Initiator](src/effects/initiator.ts) instance which has encapsulated `axios` creation and registered two interceptors automatically by default.

```ts
import { createInitiator } from './initiator'

const { http } = createInitiator(/* support all AxiosRequestConfig */)

export function fetchUserProfile(username: string, password: string) {
  return http.post('/user/profile', {
    username,
    password
  })
}
```

Based on [single responsibility principle][wiki-single-responsibility-principle] and better universality, every request route should be a singleton. All request or response errors should be handle by Http request consumer, instead of itself.

[wiki-single-responsibility-principle]: https://en.wikipedia.org/wiki/Single_responsibility_principle

## Router

|  filename   |                     description                      |
| :---------: | :--------------------------------------------------: |
| `guards.ts` | store all [navigation guards][doc-navigation-guards] |
| `index.ts`  |            export a `vue-router` instance            |
| `routes.ts` |        record all valid preset static routes         |

[doc-navigation-guards]: https://router.vuejs.org/guide/advanced/navigation-guards.html

### Route config

We add serval `meta` properties for global navigation sidebar.

```ts
interface ConsoleRouteConfig extends RouteConfig {
  meta?: Partial<{
    layout: string // which route-based layout need to be rendered
    hidden: boolean
    icon: string
    title: string
  }>
}
```

| property name |                                description                                 |
| :-----------: | :------------------------------------------------------------------------: |
| `meta.layout` | Which [route-based layout](#Use%20route-based%20layout) should be rendered |
| `meta.hidden` |       Whether route should be rendered by global navigation sidebar        |
|  `meta.icon`  |     Route `material design` icon icon in the global navigation sidebar     |
| `meta.title`  |                Route title in the global navigation sidebar                |

_Note that_ Route only be rendered when `meta.title` and `meta.hidden` is truthy value.

## State management

We use `Vuex` to implement global state management.

[doc-vuex-actions]: https://vuex.vuejs.org/guide/actions.html#actions

### Automatic registration

All store module placed in [src/store/modules/\*.ts](src/store/modules) would be **registered automatically**. Every module would use their filename as store module namespace.

### With request effects

> [Actions][doc-vuex-actions] can contain arbitrary asynchronous operations.

All HTTP requests should be called by an action if an HTTP request result needs to be stored in the vuex store, instead of calling HTTP request directly.

```
dispatch an action --> http effects --> commit mutation in an action --> store the result of effects
```

### Handle error

> [store.dispatch][doc-vuex-dispatch] always return a `Promise` instance.

[doc-vuex-dispatch]: https://vuex.vuejs.org/api/#dispatch

Any action internal error should be handled by action consumer, instead of action itself.

```
action's error --> throw --as rejected promise--> handled by any action consumer
```

### History store module

We have a `history` ([store/modules/history.ts](src/store/modules/history.ts)) store module that used to record any visited `vue-route record` with `meta.title` field.

## Changelog

All notable changes to this repository will be documented in [CHANGELOG](./CHANGELOG.md) file.

## License

MIT © [Bowen Liu](https://github.com/lbwa)

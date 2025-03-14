# swagger-axios-codegen
A swagger client uses axios and typescript


￼[![NpmVersion](https://img.shields.io/npm/v/swagger-axios-codegen.svg)](https://www.npmjs.com/package/swagger-axios-codegen)
￼[![npm](https://img.shields.io/npm/dm/swagger-axios-codegen.svg)](https://www.npmjs.com/package/swagger-axios-codegen)
[![open issues](https://img.shields.io/github/issues-raw/manweill/swagger-axios-codegen.svg)](https://img.shields.io/github/issues-raw/manweill/swagger-axios-codegen.svg)

```
< v0.16 require node > v10.0.0

>= v0.16 require node >= v16
```

it will always resolve `axios.response.data` or reject `axios.error` with Promise

support other similar to `axios` library, for example [Fly.js](https://github.com/wendux/fly), required setting `ISwaggerOptions.useCustomerRequestInstance = true`

the es6 version is generated by calling typescript

Welcome PRs and commit issue

By the way. you can support this repo via Star star sta st s... ⭐️ ⭐️ ⭐️ ⭐️ ⭐️


## [Example](./example)

## [ChangeLog](./CHANGELOG.md)

## [Contributing](./CONTRIBUTING.md)

## Get Started

```
  yarn add swagger-axios-codegen
```

```js

export interface ISwaggerOptions {
  serviceNameSuffix?: string
  enumNamePrefix?: string
  methodNameMode?: 'operationId' | 'path' | 'shortOperationId' | ((reqProps: IRequestMethod) => string)
  classNameMode?: 'parentPath' | 'normal' | ((path: string, method: string, reqProps:IRequestMethod) => string)
  /** only effect classNameMode='parentPath' */
  pathClassNameDefaultName?: string
  outputDir?: string
  fileName?: string
  remoteUrl?: string
  source?: any
  useStaticMethod?: boolean | undefined
  useCustomerRequestInstance?: boolean | undefined
  include?: Array<string | IInclude>
  /** include types which are not included during the filtering **/
  includeTypes?: Array<string>
  format?: (s: string) => string
  /** match with tsconfig */
  strictNullChecks?: boolean | undefined
  /** definition Class mode */
  modelMode?: 'class' | 'interface'
  /** use class-transformer to transform the results */
  useClassTransformer?: boolean
  /** force the specified swagger or openAPI version, */
  openApi?: string | undefined
  /** extend file url. It will be inserted in front of the service method */
  extendDefinitionFile?: string | undefined
  /** mark generic type */
  extendGenericType?: string[] | undefined
  /** generate validation model (class model mode only) */
  generateValidationModel?: boolean
  /** split request service.  Can't use with sharedServiceOptions*/
  multipleFileMode?: boolean | undefined
  /** url prefix filter*/
  urlFilters?: string[] | null | undefined
  /** shared service options to multiple service. Can't use with MultipleFileMode */
  sharedServiceOptions?: boolean | undefined
  /** use parameters in header or not*/
  useHeaderParameters?: boolean
  /** use type imports for axios to be compatible with Vite */
  useTypeImports?: boolean
}

const defaultOptions: ISwaggerOptions = {
  serviceNameSuffix: 'Service',
  enumNamePrefix: 'Enum',
  methodNameMode: 'operationId',
  classNameMode: 'normal',
  PathClassNameDefaultName: 'Global',
  outputDir: './service',
  fileName: 'index.ts',
  useStaticMethod: true,
  useCustomerRequestInstance: false,
  include: [],
  strictNullChecks: true,
  /** definition Class mode ,auto use interface mode to streamlined code*/
  modelMode?: 'interface',
  useClassTransformer: false
}

```

### use local swagger api json

```js 

const { codegen } = require('swagger-axios-codegen')
codegen({
  methodNameMode: 'operationId',
  source: require('./swagger.json')
})


```

### use remote swagger api json
```js 

const { codegen } = require('swagger-axios-codegen')
codegen({
  methodNameMode: 'operationId',
  remoteUrl:'You remote Url'
})


```

### use static method

```js
codegen({
    methodNameMode: 'operationId',
    remoteUrl: 'http://localhost:22742/swagger/v1/swagger.json',
    outputDir: '.',
    useStaticMethod: true
});

```

before


```js

import { UserService } from './service'
const userService = new UserService()
await userService.GetAll();

```

after

```js

import { UserService } from './service'

await UserService.GetAll();

```


### use custom axios.instance

```js
import axios from 'axios'
import { serviceOptions } from './service'
const instance = axios.create({
  baseURL: 'https://some-domain.com/api/',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});

serviceOptions.axios = instance

```

### use other library

```js
import YourLib from '<Your lib>'
import { serviceOptions } from './service'

serviceOptions.axios = YourLib

```

### filter service and method 

fliter by [multimatch](https://github.com/sindresorhus/multimatch) using 'include' setting

```js
codegen({
  methodNameMode: 'path',
  source: require('../swagger.json'),
  outputDir: './swagger/services',
  include: [
    '*',
    // 'Products*',
    '!Products',
    { 'User': ['*', '!history'] },
  ]
})
```

If you are using special characters in your service name (which is the first tag) you must assume they have been escaped.

For example the service names are `MyApp.FirstModule.Products`, `MyApp.FirstModule.Customers`, `MyApp.SecondModule.Orders`:

```json
// API
"paths": {
  "/Products/Get": {
    "post": {
      "tags": [
        "MyApp.FirstModule.Products"
      ],
      "operationId": "Get",
```

```js
// Codegen config
codegen({
  methodNameMode: 'path',
  source: require('../swagger.json'),
  outputDir: './swagger/services',
  include: ['MyAppFirstModule*'] // Only Products and Customers will be included. As you can see dots are escaped being contract names.
})
```


### use class transformer to transform results

This is helpful if you want to transform dates to real date 
objects. Swagger can define string formats for different types. Two if these formats are `date` and `date-time`

If a `class-transformer` is enabled and a format is set on a string, the result string will be transformed to a `Date` instance


// swagger.json
```json
{
  "ObjectWithDate": {
    "type": "object",
    "properties": {
      "date": {
        "type": "string",
        "format": "date-time"
      }
    }
  }
}
```

```js

const { codegen } = require('swagger-axios-codegen')
codegen({
  methodNameMode: 'operationId',
  source:require('./swagger.json'),
  useClassTransformer: true,
})
```

Resulting class:
```ts
export class ObjectWithDate {
  @Expose()
  @Type(() => Date)
  public date: Date;
}
```

The service method will transform the json response and return an instance of this class

### use validation model

```js
codegen({
    ...
    modelMode: 'class',
    generateValidationModel: true
});
```

The option above among with class model mode allows to render the model validation rules. The result of this will be as follows:

```js
export class FooFormVm {
  'name'?: string;
  'description'?: string;
 
  constructor(data: undefined | any = {}) {
    this['name'] = data['name'];
    this['description'] = data['description'];
  }
 
  public static validationModel = {
    name: { required: true, maxLength: 50 },
    description: { maxLength: 250 },
  };
}
```
So you can use the validation model in your application:

```js
function isRequired(vm: any, fieldName: string): boolean {
  return (vm && vm[fieldName] && vm[fieldName].required === true);
}
function maxLength(vm: any, fieldName: string): number {
  return (vm && vm[fieldName] && vm[fieldName].maxLength ? vm[fieldName].maxLength : 4000);
}
```
Now you can use the functions
```js
var required = isRequired(FooFormVm.validationModel, 'name');
var maxLength = maxLength(FooFormVm.validationModel, 'description');
```
At the moment there are only two rules are supported - `required` and `maxLength`.

## Some Solution

### 1.Reference parameters

see in [#53](https://github.com/Manweill/swagger-axios-codegen/issues/53), use package [json-schema-ref-parser](https://github.com/APIDevTools/json-schema-ref-parser)


### 2.With `Microservice Gateway`

```js
const {codegen} = require('swagger-axios-codegen')
const axios = require('axios')
// host 地址
const host = 'http://your-host-name'

// 
const modules = [
  ...
]

axios.get(`${host}/swagger-resources`).then(async ({data}) => {
  console.warn('code', host)
  for (let n of data) {
    if (modules.includes(n.name)) {
      try {
        await codegen({
          remoteUrl: `${host}${n.url}`,
          methodNameMode: 'operationId',
          modelMode: 'interface',
          strictNullChecks: false,
          outputDir: './services',
          fileName: `${n.name}.ts`,
          sharedServiceOptions: true,
          extendDefinitionFile: './customerDefinition.ts',
        })
      } catch (e) {
        console.log(`${n.name} service error`, e.message)
      }
    }
  }
})

```



원문
https://github.com/RobinBuschmann/sequelize-typescript

[![Build Status](https://travis-ci.org/RobinBuschmann/sequelize-typescript.svg?branch=master)](https://travis-ci.org/RobinBuschmann/sequelize-typescript)
[![codecov](https://codecov.io/gh/RobinBuschmann/sequelize-typescript/branch/master/graph/badge.svg)](https://codecov.io/gh/RobinBuschmann/sequelize-typescript)
[![NPM](https://img.shields.io/npm/v/sequelize-typescript.svg)](https://www.npmjs.com/package/sequelize-typescript)
[![Dependency Status](https://img.shields.io/david/RobinBuschmann/sequelize-typescript.svg)](https://www.npmjs.com/package/sequelize-typescript)

# sequelize-typescript
Decorators and some other features for sequelize (v5).

 - [Installation](#installation)
 - [Upgrade to `sequelize-typescript@1`](#upgrade-to-sequelize-typescript1)
 - [Model Definition](#model-definition)
   - [`@Table` API](#table-api)
   - [`@Column` API](#column-api)
 - [Usage](#usage)
   - [Configuration](#configuration)
   - [globs](#globs)
   - [Model-path resolving](#model-path-resolving)
 - [Model association](#model-association)
   - [One-to-many](#one-to-many)
   - [Many-to-many](#many-to-many)
   - [One-to-one](#one-to-one)
   - [`@ForeignKey`, `@BelongsTo`, `@HasMany`, `@HasOne`, `@BelongsToMany` API](#foreignkey-belongsto-hasmany-hasone-belongstomany-api)
   - [Generated getter and setter](#type-safe-usage-of-auto-generated-functions)
   - [Multiple relations of same models](#multiple-relations-of-same-models)
 - [Repository mode](#repository-mode)
   - [How to enable repository mode?](#how-to-enable-repository-mode)
   - [How to use repository mode?](#how-to-use-repository-mode)
   - [How to use associations with repository mode?](#how-to-use-associations-with-repository-mode)
   - [Limitations of repository mode](#limitations-of-repository-mode)
 - [Model validation](#model-validation)
 - [Scopes](#scopes)
 - [Hooks](#hooks)
 - [Why `() => Model`?](#user-content-why---model)
 - [Recommendations and limitations](#recommendations-and-limitations)

## Installation
*sequelize-typescript* requires [sequelize](https://github.com/sequelize/sequelize), additional typings as documented [here](http://docs.sequelizejs.com/manual/typescript.html) and [reflect-metadata](https://www.npmjs.com/package/reflect-metadata)
```
npm install sequelize
npm install @types/bluebird @types/node @types/validator
npm install reflect-metadata
```
```
npm install sequelize-typescript
```
`tsconfig.json` 에서 플래그를 아래와 같이 설정해야 합니다.
```json
"target": "es6", // 혹은 더 최근 버전의 ecmascript
"experimentalDecorators": true,
"emitDecoratorMetadata": true
```

### ⚠️ sequelize@4
`sequelize@4`는 `sequelize-typescript@0.6`을 필요로합니다. Version 0.6 [문서](https://github.com/RobinBuschmann/sequelize-typescript/tree/0.6.X) 보기.
```
npm install sequelize-typescript@0.6
```

## `sequelize-typescript@1`으로 업그레이드 하세요.
`sequelize-typescript@1` 는 `sequelize@5>=`에서만 동작합니다.  
`sequelize@4` 와 `sequelize@3`를 사용하시려면 `sequelize-typescript@0.6`을 사용하세요.

### 주요 변경 사항
`sequelize@5` 의 모든 주요 변경 사항은 `sequelize-typescript@1` 에도 유효합니다.   
v5 업그레이드에 대한 [자세한 사항](https://github.com/sequelize/sequelize/blob/master/docs/upgrade-to-v5.md).

#### 시퀄라이즈 공식 타이핑
sequelize-typescript는 이제 sequelize에 포함된 공식 타이핑을 사용합니다.
(See [this](https://github.com/sequelize/sequelize/blob/master/docs/upgrade-to-v5.md#typescript-support)).
아래와 같은 사항에 주목해주세요.
- 이전 버전에 있던 sequelize-typescript 인터페이스들은 대부분 sequelize 공식 인터페이스들로 변경되었습니다.
- `@types/sequelize`는 이제 쓰이지 않습니다.
- `@types/bluebird` is no longer an explicit dependency
- 공식 타이핑은 이전의 sequelize-typescript 문법보다 덜 엄격합니다.

#### Sequelize Options
- `SequelizeConfig` 는   `SequelizeOptions` 으로 이름이 변경되었습니다.
- `modelPaths` 속성은 `models` 으로 이름이 변경되었습니다.

#### Scopes Options
`@Scopes` 와 `@DefaultScope` 데코레이터는 이제 옵션으로 람다 함수를 갖습니다.
```ts
@DefaultScope(() => ({...}))
@Scopes(() => ({...}))
```
기존의 방식:
```ts
@DefaultScope({...})
@Scopes({...}))
```

### Repository Mode
With `sequelize-typescript@1` comes a repository mode. See [docs](#repository-mode) for details.


## Model definition
```typescript
import {Table, Column, Model, HasMany} from 'sequelize-typescript';

@Table
class Person extends Model<Person> {

  @Column
  name: string;

  @Column
  birthday: Date;

  @HasMany(() => Hobby)
  hobbies: Hobby[];
}
```
모델은 `Model`로 extned 해야 하고 `@Table` 데코레이터를 통한 주석이 달려야합니다.  
데이터베이스 컬럼에 보여야 하는 모든 속성들은 `@Column` 주석이 필요합니다.

심화 예제 보기 [클릭](https://github.com/RobinBuschmann/sequelize-typescript-example).
 
### `@Table`
`@Table` 주석은 인자 없이 사용할 수 있습니다.. 몇가지 옵션을 주기 위해서 객체 리터럴을 이용하세요. (sequelize에서 사용하는 모든 [define options](http://docs.sequelizejs.com/manual/tutorial/models-definition.html#configuration) 
은 유효합니다.):
```typescript
@Table({
  timestamps: true,
  ...
})
class Person extends Model<Person> {}
```
#### Table API

Decorator                             | Description
--------------------------------------|---------------------
 `@Table`                             | sets `options.tableName=<CLASS_NAME>` and  `options.modelName=<CLASS_NAME>` automatically
 `@Table(options: DefineOptions)`     | sets [define options](http://docs.sequelizejs.com/manual/tutorial/models-definition.html#configuration) (also sets `options.tableName=<CLASS_NAME>` and  `options.modelName=<CLASS_NAME>` if not already defined by define options) 

#### Primary key
기본키 (`id`) 가 `Model` 클래스로부터 자동으로 상속됩니다. 이 기본키는 디폴트로 `INTEGER`이고
`autoIncrement=true` 속성을 가지고 있습니다. (native sequelize와 같음). id는 다른 속성을 기본키로 설정함으로써 쉽게 재정의 될 수 있습니다. 기본키를 설정하시려면 `@Column({primaryKey: true})`을 사용하거나 `@PrimaryKey` 를 `@Column` 과 함께 사용하세요.

#### `@CreatedAt`, `@UpdatedAt`, `@DeletedAt`
Annotations to define custom and type safe `createdAt`, `updatedAt` and `deletedAt` 
attributes:
```typescript
  @CreatedAt
  creationDate: Date;

  @UpdatedAt
  updatedOn: Date;
  
  @DeletedAt
  deletionDate: Date;
```

Decorator          | Description
-------------------|---------------------
 `@CreatedAt`      | sets `timestamps=true` and `createdAt='creationDate'`
 `@UpdatedAt`      | sets `timestamps=true` and `updatedAt='updatedOn'`
 `@DeletedAt`      | sets `timestamps=true`, `paranoid=true` and `deletedAt='deletionDate'`

### `@Column`
`@Column` 주석은 인자 없이 사용될 수 있습니다. 하지만 그럴 경우 자동적으로 추론될 수 있는 js type이 필요합니다. (자세한 내용 [Type inference](#type-inference)).
```typescript
  @Column
  name: string;
```
만약 타입이 추론되지 않거나 추론되지 않아야 한다면 다음과 같이 사용하세요:
```typescript
import {DataType} from 'sequelize-typescript';

  @Column(DataType.TEXT)
  name: string;
```
혹은 더 구체적인 칼럼 정보를 위해 *sequelize*에서 유효한 오브젝트 리터럴을 사용하세요.
(all [attribute options](http://docs.sequelizejs.com/manual/tutorial/models-definition.html#configuration))
```typescript
  @Column({
    type: DataType.FLOAT,
    comment: 'Some value',
    ...
  })
  value: number;
```
#### Column API

Decorator                             | Description
--------------------------------------|---------------------
 `@Column`                            | tries to infer [dataType](http://docs.sequelizejs.com/manual/tutorial/models-definition.html#data-types) from js type
 `@Column(dataType: DateType)`        | sets [dataType](http://docs.sequelizejs.com/manual/tutorial/models-definition.html#data-types) explicitly
 `@Column(options: AttributeOptions)` | sets [attribute options](http://docs.sequelizejs.com/manual/tutorial/models-definition.html#configuration)

#### *Shortcuts*
만약 데코레이터가 너무 좋다면 *sequelize-typescript*가 제공하는 여러 데코레이터를 사용해보세요. 아래의 데코레이터를 @Column 주석과 함께 사용해서 몇가지 속성 옵션들을 쉽게 사용할 수 있습니다.

Decorator                             | Description
--------------------------------------|---------------------
 `@AllowNull(allowNull?: boolean)`    | sets `attribute.allowNull` (default is `true`)
 `@AutoIncrement`                     | sets `attribute.autoIncrement=true`
 `@Unique`                            | sets `attribute.unique=true`
 `@Default(value: any)`               | sets `attribute.defaultValue` to specified value
 `@PrimaryKey`                        | sets `attribute.primaryKey=true`
 `@Comment(value: string)`            | sets `attribute.comment` to specified string
 Validate annotations                 | see [Model validation](#model-validation)

### Type inference
아래와 같은 타입들은 자바스크립트 타입으로부터 자동으로 추론될 수 있습니다. 다른 타입들은 명시적으로 타입을 정의해야 합니다.

Design type      | Sequelize data type
-----------------|---------------------
 `string`        | `STRING`
 `boolean`       | `BOOLEAN`
 `number`        | `INTEGER`
 `Date`          | `DATE`
 `Buffer`        | `BLOB`
 
### Accessors
Get/set accessors do work as well
```typescript
@Table
class Person extends Model<Person> {

  @Column
  get name(): string {
    return 'My name is ' + this.getDataValue('name');
  }
  
  set name(value: string) {
    this.setDataValue('name', value);
  }
}
```
 
## Usage
작은 변화를 제외하고는 *sequelize-typescript*는 그냥 *sequelize*처럼 작동합니다.
(See sequelize [docs](http://docs.sequelizejs.com/manual/tutorial/models-usage.html))
### Configuration
정의한 모델을 사용하려면, `Sequelize` 인스턴스를 `sequelize-typescript`로부터 설정해야 합니다 (!).
```typescript
import {Sequelize} from 'sequelize-typescript';

const sequelize =  new Sequelize({
        database: 'some_db',
        dialect: 'sqlite',
        username: 'root',
        password: '',
        storage: ':memory:',
        models: [__dirname + '/models'], // or [Player, Team],
});
```
모델을 사용하기 전에 시퀄라이즈에게 어디에서 모델을 찾을 수 있는지 알려주어야 합니다. 따라서 sequelize config에 `models`을 설정하거나 나중에 필요한 모델을 `sequelize.addModels([Person])` 혹은 `sequelize.addModels([__dirname + '/models'])`를 사용해 추가하세요.:

```typescript
sequelize.addModels([Person]);
sequelize.addModels(['path/to/models']);
```
### globs
```typescript
import {Sequelize} from 'sequelize-typescript';

const sequelize =  new Sequelize({
        ...
        models: [__dirname + '/**/*.model.ts']
});
// or
sequelize.addModels([__dirname + '/**/*.model.ts']);
```

#### Model-path resolving
모델은 파일이름에 의해 파일과 매칭됩니다. (models나 addModels를 통해 파일을 찾은 다음에 해당 파일 내의 export 된 멤버들 중 어떤 것이 모델인지 찾는 과정)  
예제:
```typescript
// User.ts 파일은 아래와 같은 exported 모델과 매칭됩니다.
export class User extends Model<User> {}
```
매칭은 파일이름과 모든 exported 멤버들을 비교해 이루어집니다. `modelMatch` 함수를 설정에 명시해 매칭 방식을 커스터마이징 할 수 있습니다.  

예를들어 모델 이름이 `user.model.ts`이고 클래스가 `User`일때 이 둘을 다음과 같은 `modelMatch` 함수로 매칭시킬 수 있습니다.

```typescript
import {Sequelize} from 'sequelize-typescript';

const sequelize =  new Sequelize({
  models: [__dirname + '/models/**/*.model.ts']
  modelMatch: (filename, member) => {
    return filename.substring(0, filename.indexOf('.model')) === member.toLowerCase();
  },
});
```

`*.model.ts`패턴과 매칭되는 각각의 파일들에 `modelMatch`함수는 그 파일에 export 된 멤버들을 불러와 실행될 것입니다. 예를 들어 다음과 같은 파일에 대해서  

```TypeScript
//user.model.ts
import {Table, Column, Model} from 'sequelize-typescript';

export const UserN = 'Not a model';
export const NUser = 'Not a model';

@Table
export class User extends Model<User> {

  @Column
  nickname: string;
}
```
`modelMatch` 함수는 다음과 같은 인자들을 받아 3번 호출될 것입니다.
```
user.model UserN -> false
user.model NUser -> false
user.model User  -> true (User will be added as model)
```

모델을 파일과 매칭시키는 또 하나의 방법은 default export 입니다.

```TypeScript
export default class User extends Model<User> {}
```

> ⚠️모델을 추가하기 위해 paths를 사용할 경우, 그것들은 런타임에 로드된다는 것을 명심해라.
> 이것은 개발할 때와 실행할 때 path가 다를 수 있다는 것을 의미한다. 예를 들어 `.ts` 확장자를 paths에 사용할 때는 [ts-node](https://github.com/TypeStrong/ts-node)를 사용할 때만 작동한다.

### Build and create
인스턴스화(Instantiation)와 삽입(inserts)은 오래되고 좋은 시퀄라이즈 방식으로 가능합니다.
```typescript
const person = Person.build({name: 'bob', age: 99});
person.save();

Person.create({name: 'bob', age: 99});
```
하지만 *sequelize-typescript*에서는 그뿐만 아니라 `new`를 사용해 인스턴스를 생성할 수도 있습니다:
```typescript
const person = new Person({name: 'bob', age: 99});
person.save();
```

### Find and update
찾기(finding)와 업데이팅 엔트리는 순수 sequelize처럼 작동합니다. 따라서 sequelize [문서](http://docs.sequelizejs.com/manual/tutorial/models-usage.html)를 참고하세요.
```typescript
Person
 .findOne()
 .then(person => {
     
     person.age = 100;
     return person.save();
 });

Person
 .update({
   name: 'bobby'
 }, {where: {id: 1}})
 .then(() => {
     
 });
```

## Model association
관계는 `@HasMany`, `@HasOne`, `@BelongsTo`, `@BelongsToMany` 그리고 `@ForeignKey` 주석을 사용하여 모델에서 직접적으로 설명할 수 있습니다.

### One-to-many
```typescript
@Table
class Player extends Model<Player> {

  @Column
  name: string;

  @Column
  num: number;
  
  @ForeignKey(() => Team)
  @Column
  teamId: number;
  
  @BelongsTo(() => Team)
  team: Team;
}

@Table
class Team extends Model<Team> {

  @Column
  name: string;

  @HasMany(() => Player)
  players: Player[];
}
```
이게 전부입니다. *sequelize-typescript*는 나머지 모든 것을 대신 해줍니다. 그러니까 team을 `find`로 불러올 때
```typescript

Team
 .findOne({include: [Player]})
 .then(team => {
     
     team.players.forEach(player => console.log(`Player ${player.name}`));
 })
```
player 또한 resolved 됩니다. (`include: Player`를 find 옵션에 주었을 때)

### Many-to-many
```typescript
@Table
class Book extends Model<Book> { 
  @BelongsToMany(() => Author, () => BookAuthor)
  authors: Author[];
}

@Table
class Author extends Model<Author> {

  @BelongsToMany(() => Book, () => BookAuthor)
  books: Book[];
}

@Table
class BookAuthor extends Model<BookAuthor> {

  @ForeignKey(() => Book)
  @Column
  bookId: number;

  @ForeignKey(() => Author)
  @Column
  authorId: number;
}
```
#### 타입 세이프 *through*-table 객체 접근
*through*-table 객체 (instanceOf `BookAuthor` in the upper example)에 타입에 안전하게 접근하기 위해서, 타입은 수동으로 설정되어야 한다. `Author`모델에 대해서 다음과 같이 달성될 수 있다:  
To access the *through*-table instance (instanceOf `BookAuthor` in the upper example) type safely, the type 
need to be set up manually. For `Author` model it can be achieved like so:
```ts
  @BelongsToMany(() => Book, () => BookAuthor)
  books: Array<Book & {BookAuthor: BookAuthor}>;
```

### One-to-one
one-to-one 관계에서는 `@HasOne(...)`(관계를 위한 외래키가 다른 모델에 있음)과 `BelongsTo(...)`(관계를 위한 외래키가 이 모델에 있음)를 사용합니다.

### `@ForeignKey`, `@BelongsTo`, `@HasMany`, `@HasOne`, `@BelongsToMany` API

Decorator                                 | Description
------------------------------------------|---------------------
 `@ForeignKey(relatedModelGetter: () => typeof Model)` | marks property as `foreignKey` for related class 
 `@BelongsTo(relatedModelGetter: () => typeof Model)` | sets `SourceModel.belongsTo(RelatedModel, ...)` while `as` is key of annotated property and `foreignKey` is resolved from source class 
 `@BelongsTo(relatedModelGetter: () => typeof Model, foreignKey: string)` | sets `SourceModel.belongsTo(RelatedModel, ...)` while `as` is key of annotated property and `foreignKey` is explicitly specified value 
 `@BelongsTo(relatedModelGetter: () => typeof Model, options: AssociationOptionsBelongsTo)` | sets `SourceModel.belongsTo(RelatedModel, ...)` while `as` is key of annotated property and `options` are additional association options 
 `@HasMany(relatedModelGetter: () => typeof Model)` | sets `SourceModel.hasMany(RelatedModel, ...)` while `as` is key of annotated property and `foreignKey` is resolved from target related class
 `@HasMany(relatedModelGetter: () => typeof Model, foreignKey: string)` | sets `SourceModel.hasMany(RelatedModel, ...)` while `as` is key of annotated property and `foreignKey` is explicitly specified value
 `@HasMany(relatedModelGetter: () => typeof Model, options: AssociationOptionsHasMany)` | sets `SourceModel.hasMany(RelatedModel, ...)` while `as` is key of annotated property and `options` are additional association options
 `@HasOne(relatedModelGetter: () => typeof Model)` | sets `SourceModel.hasOne(RelatedModel, ...)` while `as` is key of annotated property and `foreignKey` is resolved from target related class
 `@HasOne(relatedModelGetter: () => typeof Model, foreignKey: string)` | sets `SourceModel.hasOne(RelatedModel, ...)` while `as` is key of annotated property and `foreignKey` is explicitly specified value
 `@HasOne(relatedModelGetter: () => typeof Model, options: AssociationOptionsHasOne)` | sets `SourceModel.hasOne(RelatedModel, ...)` while `as` is key of annotated property and `options` are additional association options
 `@BelongsToMany(relatedModelGetter: () => typeof Model, through: (() => typeof Model))` | sets `SourceModel.belongsToMany(RelatedModel, {through: ThroughModel, ...})` while `as` is key of annotated property and `foreignKey`/`otherKey` is resolved from through class
 `@BelongsToMany(relatedModelGetter: () => typeof Model, through: (() => typeof Model), foreignKey: string)`| sets `SourceModel.belongsToMany(RelatedModel, {through: ThroughModel, ...})` while `as` is key of annotated property, `foreignKey` is explicitly specified value and `otherKey` is resolved from through class
 `@BelongsToMany(relatedModelGetter: () => typeof Model, through: (() => typeof Model), foreignKey: string, otherKey: string)`| sets `SourceModel.belongsToMany(RelatedModel, {through: ThroughModel, ...})` while `as` is key of annotated property and `foreignKey`/`otherKey` are explicitly specified values
 `@BelongsToMany(relatedModelGetter: () => typeof Model, through: string, foreignKey: string, otherKey: string)` | sets `SourceModel.belongsToMany(RelatedModel, {through: throughString, ...})` while `as` is key of annotated property and `foreignKey`/`otherKey` are explicitly specified values
 `@BelongsToMany(relatedModelGetter: () => typeof Model, options: AssociationOptionsBelongsToMany)` | sets `SourceModel.belongsToMany(RelatedModel, {through: throughString, ...})` while `as` is key of annotated property and `options` are additional association values, including `foreignKey` and `otherKey`.

AssociationOptions를 사용할 때, 특정 속성들은 association이 built될 때 reflection metadata나 명시적 속성 인자에 의해 덮어씌워질 수 있습니다. 예를 들어 `as`는 항상 주석에 표시된 속성의 이름이 되며, `through`는 명시적으로 적힌 값이 됩니다.
 
### Multiple relations of same models
*sequelize-typescript* resolves the foreign keys by identifying the corresponding class references.

따라서 만약 다음과 같이 다수의 관계를 가진 모델을 정의한다면
```typescript
@Table
class Book extends Model<Book> { 

  @ForeignKey(() => Person)
  @Column
  authorId: number;
  
  @BelongsTo(() => Person)
  author: Person; 
  
  @ForeignKey(() => Person)
  @Column
  proofreaderId: number;
  
  @BelongsTo(() => Person)
  proofreader: Person;
}

@Table
class Person extends Model<Person> {

  @HasMany(() => Book)
  writtenBooks: Book[];

  @HasMany(() => Book)
  proofedBooks: Book[];
}
```
*sequelize-typescript* 는 어떤 외래키가 어떤 관계를 위해 사용되었는지 알지 못합니다. 따라서 외래키에 명시적으로 추가해주어야 합니다:
```typescript

  // in class "Books":
  @BelongsTo(() => Person, 'authorId')
  author: Person; 

  @BelongsTo(() => Person, 'proofreaderId')
  proofreader: Person; 
  
  // in class "Person":
  @HasMany(() => Book, 'authorId')
  writtenBooks: Book[];

  @HasMany(() => Book, 'proofreaderId')
  proofedBooks: Book[];
```

### Type safe usage of auto generated functions
관계를 생성하면서 *sequelize*는 상응하는 모델에 몇가지 메소드를 생성합니다. 따라서 `ModelA`와 `ModelB`사이에 1:n 관계를 만든다면 `ModelA`객체는 `getModelBs`, `setModelBs`, `addModelB`, `removeModelB`, `hasModelB` 함수를 가질 것입니다.이 함수들은 *sequelize-typescript*를 사용할 때에도 여전히 존재합니다. 하지만 TypeScript는 그것들을 인지하지 않고 `getModelB`, `setModelB` 나 
`addModelB`에 접근하려고 할 때 에러를 나타낼 것입니다. TypeScript가 투정부리지 않도록 *sequelize-typescript*의 `Model.prototype`는 `$set`, `$get`, `$add` 함수를 가집니다.
```typescript
@Table
class ModelA extends Model<ModelA> {

  @HasMany(() => ModelB)
  bs: ModelB[];
}

@Table
class ModelB extends Model<ModelB> {

  @BelongsTo(() => ModelA)
  a: ModelA;
}
```
그것들을 사용하기 위해서 첫번째 인자로 각각의 관계의 속성 키를 전달하세요.
```typescript
const modelA = new ModelA();

modelA.$set('bs', [ /* instance */]).then( /* ... */);
modelA.$add('b', /* instance */).then( /* ... */);
modelA.$get('bs').then( /* ... */);
modelA.$count('bs').then( /* ... */);
modelA.$has('bs').then( /* ... */);
modelA.$remove('bs', /* instance */ ).then( /* ... */);
modelA.$create('bs', /* value */ ).then( /* ... */);
```

## Repository mode
Repository mode는 `find`, `create`와 같은 정적 operation들을 모델 정의에서 분리할 수 있게 해줍니다. 또한 모델들에게 권한을 주어 모델들이 다수의 sequelize 객체들과 사용될 수 있게 해줍니다.

### How to enable repository mode?
`repositoryMode` 플래그를 셋팅해서 repository mode를 활성화할 수 있습니다:
```typescript
const sequelize = new Sequelize({
  repositoryMode: true,
  ...,
});
```
### How to use repository mode?
객체를 생성하거나 검색 작업을 수행할 저장소 검색:
```typescript
const userRepository = sequelize.getRepository(User);

const luke = await userRepository.create({name: 'Luke Skywalker'});
const luke = await userRepository.findOne({where: {name: 'luke'}});
```
### How to use associations with repository mode?
현재 관련 데이터를 검색하거나 생성하려면 include 옵션 내의 저장소를 사용해야 한다.
```typescript
const userRepository = sequelize.getRepository(User);
const addressRepository = sequelize.getRepository(Address);

userRepository.find({include: [addressRepository]});
userRepository.create({name: 'Bear'}, {include: [addressRepository]});
```
> ⚠️ 이것은 미래에 바뀔 예정이다: 저장소 대신 모델 클래스를 참조할 수 있을 것이다.

### Limitations of repository mode
Nested scopes and includes in general won't work when using `@Scope` annotation together with repository mode like:
```typescript
@Scopes(() => ({
  // includes
  withAddress: {
    include: [() => Address],
  },
  // nested scopes
  withAddressIncludingLatLng: {
    include: [() => Address.scope('withLatLng')],
  }
}))
@Table
class User extends Model<User> {}
```
> ⚠️ This will change in the future: Simple includes will be implemented.

## Model validation
Validation 옵션은 `@Column` 주석을 통해 설정될 수 있지만 따로 주석을 달아 설정하고 싶다면 다음과 같은 주석을 통해 validate option을 추가할 수 있습니다:
`validate.isEmail=true`는 `@IsEmail`이 되고, `validate.equals='value'`는 `@Equals('value')`가 되는 식입니다. boolean을 확인하는 validator는 인자가 없는 주석으로 바뀐다는 점을 신경써주세요.

See sequelize [docs](http://docs.sequelizejs.com/manual/tutorial/models-definition.html#validations) 
for all validators.

### Exceptions
다음과 같은 validator들은 sequelize validator에서 주석으로 복잡하게 바뀝니다:

Validator                        | Annotation
---------------------------------|--------------------------------------------------------
 `validate.len=[number, number]` | `@Length({max?: number, min?: number})`
 `validate[customName: string]`  | For custom validators also use the `@Is(...)` annotation: Either `@Is('custom', (value) => { /* ... */})` or with named function `@Is(function custom(value) { /* ... */})`
                                 
### Example
```typescript
const HEX_REGEX = /^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/;

@Table
export class Shoe extends Model<Shoe> {

  @IsUUID(4)
  @PrimaryKey
  @Column
  id: string;

  @Equals('lala')
  @Column
  readonly key: string;

  @Contains('Special')
  @Column
  special: string;

  @Length({min: 3, max: 15})
  @Column
  brand: string;

  @IsUrl
  @Column
  brandUrl: string;

  @Is('HexColor', (value) => {
    if (!HEX_REGEX.test(value)) {
      throw new Error(`"${value}" is not a hex color value.`);
    }
  })
  @Column
  primaryColor: string;

  @Is(function hexColor(value: string): void {
    if (!HEX_REGEX.test(value)) {
      throw new Error(`"${value}" is not a hex color value.`);
    }
  })
  @Column
  secondaryColor: string;

  @Is(HEX_REGEX)
  @Column
  tertiaryColor: string;

  @IsDate
  @IsBefore('2017-02-27')
  @Column
  producedAt: Date;
}
```

## Scopes
스코프 또한 주석으로 정의될 수 있습니다. scope 옵션은 native sequelize와 동일합니다.
(See sequelize [docs](http://docs.sequelizejs.com/manual/tutorial/scopes.html) for more details)

### `@DefaultScope` and `@Scopes`
```typescript
@DefaultScope(() => ({
  attributes: ['id', 'primaryColor', 'secondaryColor', 'producedAt']
}))
@Scopes(() => ({
  full: {
    include: [Manufacturer]
  },
  yellow: {
    where: {primaryColor: 'yellow'}
  }
}))
@Table
export class ShoeWithScopes extends Model<ShoeWithScopes> {

  @Column
  readonly secretKey: string;

  @Column
  primaryColor: string;

  @Column
  secondaryColor: string;

  @Column
  producedAt: Date;

  @ForeignKey(() => Manufacturer)
  @Column
  manufacturerId: number;

  @BelongsTo(() => Manufacturer)
  manufacturer: Manufacturer;
}
```

## Hooks
훅은 모델에 부착될 수 있습니다. 모든 모델-레벨 훅이 지원됩니다. See [the related unit tests](test/models/Hook.ts) for a summary.

각각의 hook들은 `static` 메소드이어야 합니다. 다수의 hook들은 하나의 메소드에 부착될 수 있고 다수의 메소드들을 하나의 주어진 hook에 정의할 수 있습니다.

메소드의 이름은 hook의 이름과 달라야 합니다. (예를 들어 `@BeforeCreate` hook의 메소드의 이름은 `beforeCreate`가 될 수 없습니다.) 왜냐하면 Sequelize는 hook의 이름으로 사전에 정의된 메소드들을 이미 가지고 있기 때문입니다.

```typescript
@Table
export class Person extends Model<Person> {
  @Column
  name: string;

  @BeforeUpdate
  @BeforeCreate
  static makeUpperCase(instance: Person) {
    // this will be called when an instance is created or updated
    instance.name = instance.name.toLocaleUpperCase();
  }

  @BeforeCreate
  static addUnicorn(instance: Person) {
    // this will also be called when an instance is created
    instance.name += ' 🦄';
  }
}
```

## Why `() => Model`?
`@ForeignKey(Model)` is much easier to read, so why is `@ForeignKey(() => Model)` so important? When it
comes to circular-dependencies (which are in general solved by node for you) `Model` can be `undefined`
when it gets passed to @ForeignKey. With the usage of a function, which returns the actual model, we prevent
this issue.

## Recommendations and limitations 

### One Sequelize instance per model (without repository mode)
Unless you are using the [repository mode](#repository-mode), you won't be able to add one and the same model to multiple 
Sequelize instances with differently configured connections. So that one model will only work for one connection.

### One model class per file
This is not only good practice regarding design, but also matters for the order
of execution. Since Typescript creates a `__metadata("design:type", SomeModel)` call due to `emitDecoratorMetadata` 
compile option, in some cases `SomeModel` is probably not defined(not undefined!) and would throw a `ReferenceError`.
When putting `SomeModel` in a separate file, it would look like `__metadata("design:type", SomeModel_1.SomeModel)`,
which does not throw an error.

### Minification
If you need to minify your code, you need to set `tableName` and `modelName` 
in the `DefineOptions` for `@Table` annotation. sequelize-typescript
uses the class name as default name for `tableName` and `modelName`. 
When the code is minified the class name will no longer be the originally
defined one (So that `class User` will become `class b` for example).


# Contributing

To contribute you can:
- Open issues and participate in discussion of other issues.
- Fork the project to open up PR's.
- Update the [types of Sequelize](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/sequelize).
- Anything else constructively helpful.

In order to open a pull request please:
- Create a new branch.
- Run tests locally (`npm install && npm run build && npm run cover`) and ensure your commits don't break the tests.
- Document your work well with commit messages, a good PR description, comments in code when necessary, etc.

In order to update the types for sequelize please go to [the Definitely Typed repo](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/sequelize), it would also be a good
idea to open a PR into [sequelize](https://github.com/sequelize/sequelize) so that Sequelize can maintain its own types, but that
might be harder than getting updated types into microsoft's repo. The Typescript team is slowly trying to encourage
npm package maintainers to maintain their own typings, but Microsoft still has dedicated and good people maintaining the DT repo, 
accepting PR's and keeping quality high.

**Keep in mind `sequelize-typescript` does not provide typings for `sequelize`** - these are seperate things.
A lot of the types in `sequelize-typescript` augment, refer to, or extend what sequelize already has.
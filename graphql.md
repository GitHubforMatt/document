# GraphQL

## 什么是 GraphQL

从官方的定义来说，GraphQL 是一种针对 API 的查询语言；在我看来，GraphQL 是一种标准，而与标准相对的便是实现。就像 EcmaScript 与 JavaScript 的关系，GraphQL 只定义了这种查询语言语法如何、具体的语句如何执行等。但是，你在真正使用某种 GraphQL 的服务端实现时，是有可能发现 GraphQL 标准中所描述的特性尚未被实现，例如使用node、C#、python构建GraphQL服务端时，差别会比较大，python实现字段映射的方式就感觉比node的别扭。官方网站 https://graphql.cn/

```ruby
# GraphQL查询
query{
  allBooks{
    id
  }
}


# GraphQL编辑
mutation{
  createBook(
    bookData:{
      title:"我的图书"
      author:"金庸"
    }){
    ok,
    book{
      id
    }
  }
}
```
## GraphQL有什么优势
### 声明式地数据获取
如上面看到的，GraphQL 在使用查询语句式，使用声明式的方式获取数据。客户端在一个查询请求中，选择需要的数据和相关的字段实体。客户端根据其 UI 来决定需要的字段。可以说这是 UI 驱动地数据获取。比如，需要根据不同的需求获取一个学生在移动OA、移动城管、微政务三个项目中的任意的实验数据，通常情况下我们会针对三个项目分别写一个接口，或者一个借口包含所有项目； 但使用 GraphQL在后端做好schema映射，我们可以根据需求，在一个GraphQL API里自己声明所要查询的数据进行查询，，客户端知道它需要什么数据，服务端知道数据的结构，比如如何从一些数据源（比如数据库、微服务、第三方 API）中拉取数据。

### 在 GraphQL 中没有过度获取
使用 GraphQL 不会存在过度获取的现象。使用与 Web 客户端公用的一个API，一个移动客户端很可能会获取过多的数据，但是使用同样的 GraphQL API，移动客户端可以选择和 Web 客户端不同的数据字段。GraphQL 通过最开始按客户端需求选择数据，减少了传输数据的大小。
![graphql](/images/1.jpg)


## GraphQL的缺点

1.后端或者中间层把GraphQL封装相应业务对接数据库是难点，前端使用便利则需要后端的成本来交换。后端的业务处理、字段一一映射都需要大量人力。
2.需要前端多少学一点类sql语句，不过大部分场景可以封装好固定的sql语句
封装gql不好会产生sql性能问题。


## 编写一个GraphQL服务端
### node实现GraphQL服务端
创建demo数据data.js
```ruby
let movies = [{
    id: 1,
    name: "Movie 1",
    year: 2018,
    directorId: 1
},
{
    id: 2,
    name: "Movie 2",
    year: 2017,
    directorId: 1
},
{
    id: 3,
    name: "Movie 3",
    year: 2016,
    directorId: 3
}
];

let directors = [{
    id: 1,
    name: "Director 1",
    age: 20
},
{
    id: 2,
    name: "Director 2",
    age: 30
},
{
    id: 3,
    name: "Director 3",
    age: 40
}
];

exports.movies = movies;
exports.directors = directors;
```

创建GraphQL与数据的type映射
```ruby
const {
    GraphQLObjectType,
    GraphQLID,
    GraphQLString,
    GraphQLInt,
    GraphQLList
} = require('graphql');
const _ = require('lodash');

let {movies} = require('./data.js')

movieType = new GraphQLObjectType({
    name: 'Movie',
    fields: {
        id: { type: GraphQLID },
        name: { type: GraphQLString },
        year: { type: GraphQLInt },
        directorId: { type: GraphQLID }

    }
});

directorType = new GraphQLObjectType({
    name: 'Director',
    fields: {
        id: { type: GraphQLID },
        name: { type: GraphQLString },
        age: { type: GraphQLInt },
        movies: {
            type: new GraphQLList(movieType),
            resolve(source, args) {
                return _.filter(movies, { directorId: source.id });
            }

        }

    }
});

exports.movieType = movieType;
exports.directorType = directorType;
```

编写查询
```ruby
const { GraphQLObjectType,
    GraphQLString,
    GraphQLInt
} = require('graphql');
const _ = require('lodash');

const {movieType,directorType} = require('./types.js');
let {movies,directors} = require('./data.js');

const queryType = new GraphQLObjectType({
    name: 'Query',
    fields: {
        hello: {
            type: GraphQLString,

            resolve: function () {
                return "Hello World";
            }
        },

        movie: {
            type: movieType,
            args: {
                id: { type: GraphQLInt }
            },
            resolve: function (source, args) {
                return _.find(movies, { id: args.id });
            }
        }

        ,

        director: {
            type: directorType,
            args: {
                id: { type: GraphQLInt }
            },
            resolve: function (source, args) {
                return _.find(directors, { id: args.id });
            }
        }
    }
});

exports.queryType = queryType;
```

最后创建graphql服务，这是我使用的是比价简单常用的express框架做demo
```ruby
const express = require('express');
const graphqlHTTP = require('express-graphql');
const {GraphQLSchema} = require('graphql');

const {queryType} = require('./query.js');

const port = 5000;
const app = express();

const schema = new GraphQLSchema({ query: queryType });

app.use('/graphql', graphqlHTTP({
    schema: schema,
    graphiql: true,
}));

app.listen(port);
console.log(`GraphQL Server Running at localhost:${port}`);
```

执行node server.js创建服务，打开http://localhost:5000/graphql 即可看到graphql交互界面。
```ruby
node server.js
```
![graphql](/images/3.jpg)

### python以django为例
创建model
```ruby
class Book(models.Model):
    title = models.ForeignKey(Title, related_name='book_title', on_delete=models.CASCADE)
    author = models.ForeignKey(Author, related_name='book_author', on_delete=models.CASCADE)
```

定义model与graphql字段映射
```ruby
class BookType(DjangoObjectType):
    class Meta:
        model = Book
```

定义查询输入
```ruby
# 定义动作，类似POST, PUT, DELETE
class BookInput(graphene.InputObjectType):
    title = graphene.String(required=True)
    author = graphene.String(required=True)
```

添加create方法
```ruby
class CreateBook(graphene.Mutation):
    # api的输入参数
    class Arguments:
        book_data = BookInput(required=True)

    # api的响应参数
    ok = graphene.Boolean()
    book = graphene.Field(BookType)

    # api的相应操作，这里是create
    def mutate(self, info, book_data):
        title = Title.objects.create(title=book_data['title'])
        author = Author.objects.create(name=book_data['author'])
        book = Book.objects.create(title=title, author=author)
        ok = True
        return CreateBook(book=book, ok=ok)
```

添加查询
```ruby
class Query(object):
    all_books = graphene.List(BookType)

    def resolve_all_books(self, info, **kwargs):
        # 查询所有book的逻辑
        return Book.objects.all()
```

在schema文件中实现Query、Mutations
```ruby
class Query(book.schema.Query, graphene.ObjectType):
    # 总的Schema的query入口
    pass

class Mutations(graphene.ObjectType):
    # 总的Schema的mutations入口
    create_book = book.schema.CreateBook.Field()

schema = graphene.Schema(query=Query, mutation=Mutations)
```

## 关于GraphQL的个人总结
我理解的GraphQL本质是一个标准，一个api的表现形式，但是它在功能是我们可以把它理解为一个类似orm的东西，介于数据和业务之间，将分块不同源的数据整合，统一输出。使用它可以很好的解决前后的协同问题，api的修改、版本维护问题通通迎刃而解，但是它对于后端的工作量要求较高，需要较多的数据映射和业务处理。另外node版本示例非常简单，阅读全文直接拷贝代码就可体验GraphQL。


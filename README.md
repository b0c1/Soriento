Soriento
========

[![Join the chat at https://gitter.im/dimparf/Soriento](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/dimparf/Soriento?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Build Status](https://travis-ci.org/b0c1/Soriento.svg)](https://travis-ci.org/b0c1/Soriento)

## Scala OrientDb object mapping library

Soriento is an object-relational mapping framework from scala case classes to OrientDb ODocument.
##News
 - Support LinkSet, LinkList annotated fields

## Features

 - Creating/deleting OrientDb classes by case classes.
 - Transparent CRUD for documents represented as case classes.
 - Linked or Embedded definitions for case classes.
 - Support serialization/deserialization for case classes with @Embedded list/set of case classes.
 - Support OType mapping Scala => OrientDb OTypes.
 - Transactions support.
 - Query by SQL.
 
Supported types:
- Long, Double, Int, Float, Short, String, case classes: Embedded, EmbeddedList, EmbeddedSet, Link, LinkList, LinkSet

##Coming soon
- LinkMap.
- EmbeddedMap.

##Add to you project
```scala
 lazy val youProject = Project("YouProject", file("."))
  .settings(commonSettings: _*)
  .dependsOn(sorientoProject)
  
 lazy val sorientoProject = RootProject(uri("https://github.com/dimparf/Soriento.git#master"))
```

##Usage
Simple example:
```scala
  import com.emotioncity.Dsl._ // or extends Dsl trait
  
  implicit val orientDb: ODatabaseDocumentTx = ???

  //create case classes with ODocumentReader for it
  case class Message(content: String)
  object Message {
    implicit object MessageReader extends ODocumentReader[Message] {
      def read(oDocument: ODocument): Message = {
        Message(
          oDocument.get[String]("content").get
        )
      }
    }
  }
  
  case class Blog(author: String, @Embedded message: Message) // or @Linked
  object Blog {
     implicit object BlogReader extends ODocumentReader[Blog] {
       def read(oDocument: ODocument): Blog = {
         Blog(
           oDocument.get[String]("author").get,
           oDocument.getAs[Message]("message").get
         )
       }
     }
  }
  
  case class BlogWithEmbeddedMessages(author: String, @EmbeddedSet messages: List[Message])
  object BlogWithEmbeddedMessages {
     implicit object BlogWithEmbeddedMessagesReader extends ODocumentReader[BlogWithEmbeddedMessages] {
        def read(oDocument: ODocument): BlogWithEmbeddedMessages = {
           BlogWithEmbeddedMessages(
             oDocument.get[String]("author").get,
             oDocument.getAs[Message, List[Message]]("message").get
           )
        }
     }
  }

  
  //schema-full (use com.emotioncity.soriento.ODb trait) mode or without this lines - schema less
  createOClass[Message] 
  createOClass[Blog]
  createOClass[BlogWithEmbeddedMessages]
  
  val blog = Blog("Dim", message = Message("Hi")) //or without named params Blog("Dim", Message("Hi))
  val blogWithEmbeddedMessages = BlogWithEmbeddedMessages("John", List(Message("Hi"), Message("New blog note")))
  //ActiveRecord style
  blog.save
  blogWithEmbeddedMessages.save
  
  //..or
  orientDb.save(blog)
  
  
  val blogs: List[Blog] = db.queryBySql[Blog]("select from blog")
  
  //Save object graph (from test code, use scalatest)
  val messageOne = LinkedMessage("This is my first message")
  val messageOneSaved = messageOne.save.as[LinkedMessage].get
  val messageTwo = LinkedMessage("last")
  val messageTwoSaved = messageTwo.save.as[LinkedMessage].get
  
  //Warning: Soriento use immutable case classes, unsaved messages don't have id. Save your values and get saved object with id with as[T] method.
  val blogWithLinkSetMessages = BlogWithLinkSetMessages("MyBlog", Set(messageOneSaved, messageTwoSaved))
  blogWithLinkSetMessages.save
  
  val extractedBlogsOpt = orientDb.queryBySql[BlogWithLinkSetMessages]("select from BlogWithLinkSetMessages where name = 'MyBlog'").headOption
    extractedBlogsOpt match {
      case Some(extractedBlog) =>
        inside(extractedBlog) { case BlogWithLinkSetMessages(name, messages) =>
          name should equal("MyBlog")
          messages should have size 2
          messages should contain(LinkedMessage("This is my first message", messageOneId))
          messages should contain(LinkedMessage("last", messageTwoId))
        }
      case None => fail("Model not saved or retrieved")
    }
  }
    
  deleteOClass[Message]
  deleteOClass[Blog]
  deleteOClass[BlogWithEmbeddedMessages]
  deleteOClass[BlogWithLinkSetMessages]
```

More examples in test directory.

## Testing
To run unit tests:

    sbt test

## Contributing

Welcome to contribute!
You can always post an issue or (even better) fork the project, implement your idea or fix the bug you have found and send a pull request. 
Just remember to test it when you are done. Here is how:

Run sbt test to run and compile tests.
    
## License
This software is available under the [Apache License, Version 2.0](LICENSE).    

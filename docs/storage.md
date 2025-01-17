# Storage

Firebase comes with many features, it is not limited to 
authentication. This document covers 
[Firebase Storage](https://firebase.google.com/docs/storage),
a storage system akin to AWS S3 though more flexible.

Firebase Storage stores files, you can add files, download files,
delete files, and list the files. These files can be in buckets,
or directories.

This requires the authentication to work, which has the advantage
of easily ensuring that only those who may access certain files can.

!!! tip "Permissions"
    Note the firebase implements everything from the front-end:
    users of your shiny application will access the storage bucket
    via _their own firebase credentials._

Let's start from a basic application that features authentication.
The app below uses the pre-built UI, probably the simplest app
that can be made using Firebase.

```r
library(shiny)
library(firebase)

ui <- fluidPage(
  useFirebase(), # import dependencies
  firebaseUIContainer(),
  reqSignin(h4("Logged in!")) # hide from UI
)

server <- function(input, output){
  f <- FirebaseUI$
    new()$ # instantiate
    set_providers( # define providers
      email = TRUE, 
      google = TRUE
    )$
    launch() # launch
}

shinyApp(ui, server)
```

## Initialise

To use the storage we need to Initialise it client-side, it's
very simple.

```r
library(shiny)
library(firebase)

ui <- fluidPage(
  useFirebase(), # import dependencies
  firebaseUIContainer(),
  reqSignin(h4("Logged in!")) # hide from UI
)

server <- function(input, output){
  f <- FirebaseUI$
    new()$ # instantiate
    set_providers( # define providers
      email = TRUE, 
      google = TRUE
    )$
    launch() # launch

  s <- Storage$new() # initialise
}

shinyApp(ui, server)
```

## Reference

A crucial concept with the `Storage` is the idea of the 
reference. The reference is the path, directory, or bucket
one is currently working with.

This is defined by a method on its own, methods used subsequently
will then use this as reference.

For instance one can reference the file `test.png` even if it does
not exist, then calling the method to upload a file will upload 
the file as `test.png`. _Make sure you keep track of this._

## Response

Much of JavaScript works asynchronously, it's certainly one of its
strong points. This means that when a function is called we do not
receive the result from said function straight away, we get a promise
back. Exactly like the R {promises} package.

Hence when triggering a sign in with other classes of the package
the response is not returned by the same function. This is because
the results of the authentication are returned later, and we cannot
know when.

The same applies to the storage: when we upload a file we cannot 
know whether the upload was successful at the time of the upload,
only later, and so on for every action we take on the storage.

Therefore, upon performing such actions (upload, delete, etc.)
we (optionally) specify a `response`. This response identifier
can then be used with the `get_response` method to retrieve 
the results of said operation: `get_response` acts exactly like
any other shiny `input`.

## Upload

We can then upload a file with the `upload_file` method.

Note that below we use a file on disk but you can use uploaded
files: the method accepts the path to a file which can be obtained
from a file upload in Shiny.

We also set the reference (`ref`) to `test.png`, we'll be working with this
file; it does not exist yet, we'll create it, download it, delete it,
etc.

The button to upload is only rendered if the user is logged in.
The button triggers to `upload_file` method that uploads a file
from the disk. To this method we also pass the name of the input
we want to set to capture the response.

Below we set the response of the upload to `up` so we can pick
up the response with `input$up`.

Note that you can set `response` to `FALSE` if you do not want
to retrieve the results.

```r
library(shiny)
library(firebase)

ui <- fluidPage(
  useFirebase(), # import dependencies
  firebaseUIContainer(),
  reqSignin(h4("Logged in!")), # hide from UI
  uiOutput("uploadUI")
)

server <- function(input, output){
  f <- FirebaseUI$
    new()$ # instantiate
    set_providers( # define providers
      email = TRUE, 
      google = TRUE
    )$
    launch() # launch

  s <- Storage$new()$
    ref("test.png")
  
  # upload a file
  output$uploadUI <- renderUI({
    f$req_sign_in()

    actionButton("upload", "Upload Image")
  })

  observeEvent(input$upload, {
    s$upload_file("/path/to/file.png", "up")
  })

  observeEvent(input$up, {
    print(input$up)
  })
}

shinyApp(ui, server)
```

## Download

Once the file uploaded we can add a button to download the file.
It does not truly download the file but will retrieve a valid URL
to it. You may then do what you want with said link, download the file
with {httr} (or {httr2}), use it as `src` atribute for a an `<img/>`
tag, etc.

```r
library(shiny)
library(firebase)

ui <- fluidPage(
  useFirebase(), # import dependencies
  firebaseUIContainer(),
  reqSignin(h4("Logged in!")), # hide from UI
  uiOutput("uploadUI"),
  uiOutput("downloadUI")
)

server <- function(input, output){
  f <- FirebaseUI$
    new()$ # instantiate
    set_providers( # define providers
      email = TRUE, 
      google = TRUE
    )$
    launch() # launch

  s <- Storage$new()$
    ref("test.png")
  
  # upload a file
  output$uploadUI <- renderUI({
    f$req_sign_in()

    actionButton("upload", "Upload Image")
  })

  observeEvent(input$upload, {
    s$upload_file("/path/to/file.png", "up")
  })

  observeEvent(input$up, {
    print(input$up)
  })
  
  # download a file
  output$downloadUI <- renderUI({
    f$req_sign_in()

    actionButton("download", "Download Image")
  })

  observeEvent(input$download, {
    s$download_file("dl")
  })

  observeEvent(input$dl, {
    print(input$dl)
  })
}

shinyApp(ui, server)
```

## Delete

To delete a file simply call the `delete_file` method.

```r
library(shiny)
library(firebase)

ui <- fluidPage(
  useFirebase(), # import dependencies
  firebaseUIContainer(),
  reqSignin(h4("Logged in!")), # hide from UI
  uiOutput("uploadUI"),
  uiOutput("downloadUI"),
  uiOutput("deleteUI")
)

server <- function(input, output){
  f <- FirebaseUI$
    new()$ # instantiate
    set_providers( # define providers
      email = TRUE, 
      google = TRUE
    )$
    launch() # launch

  s <- Storage$new()$
    ref("logo.png")
  
  # upload a file
  output$uploadUI <- renderUI({
    f$req_sign_in()

    actionButton("upload", "Upload Image")
  })

  observeEvent(input$upload, {
    s$upload_file("docs/logo.png", "up")
  })

  observeEvent(input$up, {
    print(input$up)
  })
  
  # download a file
  output$downloadUI <- renderUI({
    f$req_sign_in()

    actionButton("download", "Download Image")
  })

  observeEvent(input$download, {
    s$download_file("dl")
  })

  observeEvent(input$dl, {
    print(input$dl)
  })
  
  # delete file
  output$deleteUI <- renderUI({
    f$req_sign_in()

    actionButton("delete", "Delete Image")
  })

  observeEvent(input$delete, {
    s$delete_file("del")
  })

  observeEvent(input$del, {
    print(input$del)
  })
}

shinyApp(ui, server)
```

## Other Methods

There are other methods to list files, and retrieve metadata but
they work exactly like all others so we'll live it at that

# Hosting on the Web without the Fluff




## Prelude
I have a simple rust application that I want to host on the web that can be found [here](https://github.com/NitroSniper/ortin.git). 

main.rs
```rust
use askama_axum::Template;
use axum::{routing::get, Router};
use tokio::fs;
use tower_http::services::ServeDir;

#[derive(Template)]
#[template(path = "blog.html")]
struct BlogTemplate {
    content: String,
}

#[derive(Template)]
#[template(path = "home.html")]
struct HomeTemplate<'a> {
    dark: bool,
    text: &'a str,
}

#[tokio::main]
async fn main() {
    let router = Router::new()
        .route(
            "/",
            get(|| async {
                HomeTemplate {
                    dark: false,
                    text: "# Hello World",
                }
            }),
        )
        .route(
            "/blog",
            get(|| async {
                BlogTemplate {
                    content: markdown::to_html(
                        &fs::read_to_string("src/pi.md")
                            .await
                            .expect("to be a markdown file"),
                    ),
                }
            }),
        )
        .nest_service("/static", ServeDir::new("static"))
        .layer(tower_livereload::LiveReloadLayer::new());

    let listener = tokio::net::TcpListener::bind("0.0.0.0:8000").await.unwrap();
    axum::serve(listener, router).await.unwrap();
}
```
This is great but it is only hosted locally. I want this published on the web so first we need to setup a server. 

## Setting up a Server
I chose NixOS because it robust and minimal and I will be hosting this on a RasberryPi

### Installing NixOS on RasberryPi








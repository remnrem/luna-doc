# Running Hypnoscope

## remnrem.net

A version is hosted online at
[https://remnrem.net](https://remnrem.net).  For uploading large
`.hypnos` files, it will be better to run Hypnoscope locally, however.

## Dockerized version

The [Dockerized](download/docker.md) version of Luna contains Hypnoscope.

1) Start the Docker container (described [here](download/docker.md#pulling-luna) and [here](download/docker.md#lunadocker) and go to the `hypnoscope`
folder, and make sure it is up-to-date:

```
cd /build/hypnoscope/
git pull
```

2) Start R and launch  _Hypnoscope_:

```
library(shiny)
runApp( launch.browser=F , host = "0.0.0.0" , port = 3838 )
```

3) Visit your local browser at [127.0.0.1:3838]()


## Running locally

Obtain Hypnoscope
```
git clone https://github.com/remnrem/hypnoscope.git
cd hypnoscope
```

In R:
```
library(shiny)
runApp()
```
which should bring up a browser window.

_Hypnoscope_ will be distributed as a proper R package, but for now,
you may need to first install these libraries (as well as having
[lunaR](ext/R/index.md) installed):

```
install.packages( c("shiny","shinybusy","shinyjs","dplyr","lubridate","shinydashboard","datamods") )
```

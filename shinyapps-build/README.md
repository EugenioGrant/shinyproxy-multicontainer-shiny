
# shinyapps R package with two Shiny apps
Contains two Shiny apps:
* `run_07_widgets`
* `run_04_mpg`


## Characteristics
* Application bundled as a package `shinyapps`
* This container only deals with the Shiny apps
* ShinyProxy has to be launched in another container


## Build the image
To build the image from the Dockerfile, navigate into the root directory of this repository and run

```
docker build -t shinyapps-img .
```

## Run a container
Optionally, for testing, you may run the container with:

```
docker run -it --rm -p 3838:3838 --name shinyapps-k shinyapps-img
```

Stop it after you finish your testing.
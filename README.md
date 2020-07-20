

[toc]





# Multi-container Shiny Applications with ShinyProxy. Example 2

## Run a R package with two Shiny apps as a container

The build files will be located in the folder `shinyapps-build`. Originally this was a ShinyProxy demo but the names were little bit confusing so I renamed the package from `shinyproxy` to `shinyapps`, the Shiny applications, loaded different Shiny demos, used different  names for the image and the R package itself. I hope this example is  clear enough to understand running multiple containers from *ShinyProxy*.

Again, I will be using *VS-code* as my editor and terminal provider. There are three important folders:



![Image for post](https://miro.medium.com/max/319/1*gsEdeEULh1pso9JQtLbjvA.png)

-   `euler`: the previous containerized Shiny application
-   `shinyapps-build`: a R package with two Shiny apps that will be containerized
-   `shinyproxy-container`: ShinyProxy Java executable that will be containerized as well

### Build the image for the R package `shinyapps`

The R package `shinyapps` contains two small Shiny apps: `run_07_widgets()` and `run_04_mpg()`, both come with the *Shiny* package as examples. This is the R code for both apps:



![Image for post](https://miro.medium.com/max/684/1*uF2YUjoE2P0ERXkD46U3Xg.png)

The code is available in the repository. Pasting the image here looks better than pasting the code as text.

Right-click on the folder `shiny-apps-build` and open a terminal. See the image below.



![Image for post](https://miro.medium.com/max/642/1*xpa8XJGX2x2t4NVQN6zm9A.png)

The Dockerfile looks like this:

```
FROM openanalytics/r-baseMAINTAINER Tobias Verbeke "tobias.verbeke@openanalytics.eu"RUN apt-get update && apt-get install -y \
    sudo \
    pandoc \
    pandoc-citeproc \
    libcurl4-gnutls-dev \
    libcairo2-dev \
    libxt-dev \
    libssl-dev \
    libssh2-1-dev \
    libssl1.0.0# packages needed for basic shiny functionality
RUN R -e "install.packages(c('shiny', 'rmarkdown'), repos='https://cloud.r-project.org')"# install shinyapps package with demo shiny application
COPY shinyapps_0.0.2.tar.gz /root/
RUN R CMD INSTALL /root/shinyapps_0.0.2.tar.gz
RUN rm /root/shinyapps_0.0.2.tar.gz# set host and port
COPY Rprofile.site /usr/lib/R/etc/EXPOSE 3838CMD ["R", "-e", "shinyapps::run_07_widgets()"]
```

Build the image with:

```
docker build -t shinyapps-img .
```

We can quick test the container with:

```
docker run -it --rm -p 3838:3838 --name shinyapps-k shinyapps-img
```

And watch one of the apps in the browser:



![Image for post](https://miro.medium.com/max/875/1*K2oY_evYwXHERsiANerGRA.png)

Interrupt the container with `Ctrl-C` in the terminal. Although, we see one application here, this package  contains two Shiny apps. We just can’t see them here until we launch *ShinyProxy*.

Next, we add these two Shiny apps to the *ShinyProxy* applications file `application.yml`.

### Run ShinyProxy as a container

Now, let’s right-click on the folder `shinyproxy-container` in *vscode* and open a terminal. See the image below:



![Image for post](https://miro.medium.com/max/608/1*2a0MRuWVwRxloTdUiNTdZQ.png)

Open the file `application.yml` and see how we added the two Shiny apps:



![Image for post](https://miro.medium.com/max/704/1*3TWZHAXeRQSZuEMhVGJ0zw.png)

Also observe that there are two significant changes compared with the first example: we have `internal-networking: true`, and we added a network for the containers `container-network: sp-example-net`. This how Docker commands get passed between *ShinyProxy* and the containers.

Another thing that will be different is that we will be containerizing *ShinyProxy*; it will not run from the host or physical machine but from a container based on the following image `Dockerfile`:which is very simple: its function is provide a Java environment to be able to run the ShinyProxy `jar` file from within.



![Image for post](https://miro.medium.com/max/796/1*UGuYofoQ25T_qxpc0YA9kA.png)

Let’s build the image with:

```
docker build -t shinyproxy2-img .
```



![Image for post](https://miro.medium.com/max/649/1*JBDi_G3OTTtLJERnMcUh-w.png)

Note. I am using the number 2 in `shinyproxy2-img` to mean that it will be handling 2 Shiny apps.

Now, it’s time to see all these containers working together. We run the ShinyProxy container with:

```
docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    --net sp-example-net \
    -p 8080:8080 \
     shinyproxy2-img
```

*ShinyProxy* is up and running now in its own container:



![Image for post](https://miro.medium.com/max/892/1*tkmTS99iQX3UeHCqF6I_5Q.png)

We go see check `csysdig` and we found one container running:



![Image for post](https://miro.medium.com/max/1087/1*vBFSdOB91VCudX9e61E0CQ.png)

Let’s open a browser with:

```
127.0.0.1:8080
```

We now see two applications in the dashboard:



![Image for post](https://miro.medium.com/max/926/1*909IivMpaEwcXLP0QlCQKA.png)

We launch one of the apps **MPG Application**:



![Image for post](https://miro.medium.com/max/906/1*UcFbblh1aS5jZxYhE7xFZQ.png)

From `csysdig` we now can see two active containers; one based on the image `shinyapps-img`, and the second from `shinyproxy2-img`.



![Image for post](https://miro.medium.com/max/1163/1*HJcIXJ8VwpV5ZJtWNYiB-g.png)

Let’s stop everything by interrupting *ShinyProxy* with `Ctrl-C`. Back to `csysdig`, all the containers have been stopped.


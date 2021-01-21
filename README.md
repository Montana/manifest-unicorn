![Manifest](manifest.gif)

## Intro:

The Docker container `manifest` is a file that contains data about a container image. Specifically `digest`, `sha256`, etc. We can create a `manifest` which points to images for different architectures so that when using the image on a particular architecture the docker automatically pulls the desired image.

Creating a ```manifest``` is an experimental CLI feature and you should update docker for good measure. Add following lines to .travis.yml:

```yaml
addons:
  apt:
    packages:
      - docker-ce
```
Below is a sample of a `manifest` that's being automated:

```bash
export DOCKER_CLI_EXPERIMENTAL=enabled # crucial for manifest to work 

docker manifest create IBM/ibm-image:latest \
            IBM/ibm-image:latest-${PLATFORM_1} \ # arch s390x 
            IBM/ibm-image:latest-${PLATFORM_2} \ # arch ppc64le
            
docker manifest annotate someone/my-image:latest someone/my-image:latest-${PLATFORM_1} --arch ${PLATFORM_1}
docker manifest annotate someone/my-image:latest someone/my-image:latest-${PLATFORM_2} --arch ${PLATFORM_2}

docker manifest push IBM/ibm-image:latest
```
> You've just read an automated creation and push of `manifests`. 

Then you're ready to go. 

### Setting up your Docker Env Vars:

You should see this in your build at some point, this is reassurance your `env vars` got saved.

![envvars](dockervars.png)

> Travis CI confirming that our `env vars` are safe and secure while it builds. 

For some setting the `env vars` in the CLI is the best option, but for others using the Travis CI user interface is easier, and quicker. Click Settings -> Environment Variables then add your `DOCKER` env vars. 

![UI](envvarui.png)

> We are using the UI to enter the Travis CI env vars (rather than the CLI).

### Your `.travis.yml` setup (the most important parts): 

```yaml
---
install:
  - docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

before_script:
  - export ver=$(curl -s "https://pkgs.alpinelinux.org/package/edge/main/x86_64/curl" | grep -A3 Version | grep href | sed 's/<[^>]*>//g' | tr -d " ")
  - export build_date=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
  - export vcs_ref=$(git rev-parse --short HEAD)

  # Montana's crucial workaround
  
script:
  - chmod u+x ./travis.sh
  - export DOCKER_CLI_EXPERIMENTAL=enabled
  - chmod u+x ./build.sh

after_success:
  - docker images
  - docker manifest inspect --verbose ppc64le/node
  - docker manifest inspect --insecure ppc64le/node
  - docker manifest inspect --verbose s390x/python
  - docker manifest inspect --insecure s390x/python
  - docker manifest inspect --verbose ibmjava:jre
  - docker manifest inspect --insecure ibmjava:jre

branches:
  only:
    - master
  except:
    - /^*-v[0-9]/
    - /^v\d.*$/
```

Once we've created our `env vars`  `DOCKER_USERNAME` and `DOCKER_PASSWORD` you can start using `docker manifest`. 





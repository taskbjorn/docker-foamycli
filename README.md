> [!WARNING]
> **ARCHIVED REPOSITORY**
>
> This project is deprecated, and this repository is available only for
> archiving purposes. I am also no longer maintaining the associated Docker
> image, which is probably wildly outdated by now.

# Quick reference

* **Maintained by:** [taskbjorn](https://github.com/taskbjorn/foamycli)

* **Where to get help:** [GitHub](https://github.com/taskbjorn/foamycli/issues)

# Supported tags and respective Dockerfile links

* [**OpenFOAM**](https://github.com/taskbjorn/foamycli/build/vanilla) (OpenFOAM
  Foundation Inc.)
  * `latest-vanilla`, `of9`, `9`.
* [**OpenFOAM**](https://github.com/taskbjorn/foamycli/build/plus) (OpenCFD
  Ltd.)
  * `latest-plus`, `plus-v2106`, `v2106`.
* [**FOAM-Extend**](https://github.com/taskbjorn/foamycli/build/extend) (Wikki
  Ltd.)
  * `latest-extend`, `fe4.1`, `4.1`.

# What is `foamycli`?

`foamycli` is a minimal Docker container for OpenFOAM.

![foamycli-logo](./docker-foamycli.png)

The container is based on a minimal Debian image and only includes command-line
tools. It does not include Paraview, as it is meant to be used on a Windows host
using the native version of ParaView for postprocessing.

OpenFOAM is compiled with the following third-party tools in addition to the
ones included with the
[ThirdParty](https://openfoam.org/download/source/third-party-software/) release
repo of the chosen version:

* [CGAL](https://www.cgal.org).
* [Metis](http://glaros.dtc.umn.edu/gkhome/views/metis).

The following system tools are added to the Debian Slim image:

* Zsh shell with [Oh My Zsh](https://ohmyz.sh).
* [tmux](https://github.com/tmux/tmux/wiki), a terminal multiplexer.
* [htop](https://htop.dev), an interactive process viewer.
* [Ranger](https://github.com/ranger/ranger), a console file manager with VI key
  bindings.
* [Python 3](https://www.python.org) with package installer
  [pip](https://pypi.org/project/pip/).
* [pyFoam](https://pypi.org/project/PyFoam/), a library to support working with
  OpenFOAM.

# How to use this image

Sample Compose files are located under [./docker/compose](). An example for the
vanilla version, permanently storing case files under a Docker volume named
`openfoam_cases`, is provided below. Substitute `{HOST-IP}` with the IP of the
Docker host running an X server if you require graphica functionality from
inside the container (e.g. `gnuplot` output).

```yml
version: "3.7"

services:
  openfoam:
    container_name: openfoam
    environment:
      - "DISPLAY={HOST-IP}:0.0"
    image: taskbjorn/foamycli:latest
    network_mode: none
    user: 1000:1000
    volumes:
      - cases:/home/openfoam/Cases

volumes:
  cases:
    name: openfoam_cases
```

# Caveats

The container runs under user `openfoam` (UID 1000) and group `openfoam` (GID
1000). The OpenFOAM build is stored under `/opt/OpenFOAM` with permissions
`775`. It is convenient to store cases under `~/Cases` as all images
(`foamycli`, `foamycli-plus` and `foamycli-extend`) run under the same user,
making it easy to share case files between containers without facing permissions
issues.

# License

This image is licensed under [GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)
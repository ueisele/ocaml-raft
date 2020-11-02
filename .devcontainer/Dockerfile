FROM ocaml/opam2:debian-9-opam

USER root

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update -y \
    && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends \
        bash zsh wget curl git gnupg2 watch procps \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

ARG PYTHON_VERSION=3.9.0
RUN curl -o ~/python-debian.sh https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/python-debian.sh \
    && chmod +x ~/python-debian.sh \
    && ~/python-debian.sh ${PYTHON_VERSION} /usr/local/python/${PYTHON_VERSION} /usr/local/py-utils opam "true" "true" \
    && rm -f ~/python-debian.sh \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*
ENV PIP_TARGET=/usr/local/pip-global
RUN mkdir -p ${PIP_TARGET} && chown opam:root ${PIP_TARGET}
ENV PYTHONPATH=${PIP_TARGET}

ARG NODEJS_VERSION=15.0.1
ENV NVM_DIR=/usr/local/share/nvm
RUN curl -o ~/node-debian.sh https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/node-debian.sh \
    && chmod +x ~/node-debian.sh \
    && ~/node-debian.sh ${NVM_DIR} ${NODEJS_VERSION} opam "true" \
    && rm -f ~/node-debian.sh \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*
RUN . ${NVM_DIR}/nvm.sh && npm i -g eslint

RUN apt-get update -y \
    && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends \
        gcc g++ make \
        m4 \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

ENV DEBIAN_FRONTEND=

USER opam

RUN mkdir -p /home/opam/bin && mkdir -p /home/opam/.local/bin
ENV PATH=/home/opam/bin:/home/opam/.local/bin:${PATH}

ARG OCAML_VERSION=4.02.3
RUN opam-sandbox-disable \
    && cd /home/opam/opam-repository && git pull \
    && opam init -k git -a /home/opam/opam-repository --bare \
    && opam switch create ${OCAML_VERSION} ocaml-base-compiler.${OCAML_VERSION} \
    #&& opam switch create ${OCAML_VERSION}+musl ocaml-variants.${OCAML_VERSION}+musl \
    #&& opam switch create ${OCAML_VERSION}+musl+static ocaml-variants.${OCAML_VERSION}+musl+static
    #&& opam switch create ${OCAML_VERSION}+afl ocaml-variants.${OCAML_VERSION}+afl \
    #&& opam switch create ${OCAML_VERSION}+flambda ocaml-variants.${OCAML_VERSION}+flambda \
    && opam switch ${OCAML_VERSION} \
    && opam install -y depext
ENV OPAMYES 1
ENTRYPOINT [ "opam", "config", "exec", "--" ]
CMD bash

RUN cd /home/opam/opam-repository \
    && git pull \
    && opam update \
    && opam upgrade \
    && opam install \
        oasis ocamlfind \
        #oasis.0.4.5
        core.113.00.00 \
        #core_kernel camlp4 sexplib bin_prot fieldslib variantslib yojson comparelib \
        ounit qcheck \
        js_of_ocaml js_of_ocaml-compiler js_of_ocaml-ppx

#RUN opam install ocamlformat ocaml-lsp-server
RUN . ${NVM_DIR}/nvm.sh && npm i -g bash-language-server

# Specific glibc: https://stackoverflow.com/questions/2856438/how-can-i-link-to-a-specific-glibc-version

FROM opnfv/functest-core

ARG BRANCH=master
ARG OPENSTACK_TAG=stable/queens
ARG VIMS_TEST_TAG=stable
ARG VIMS_TAG=fraser
ARG VROUTER_TAG=fraser
ARG JUJU_TAG=tags/juju-2.2.5

ENV GOPATH /src/epc-requirements/go
ENV GOBIN /src/epc-requirements/go/bin
ENV PATH $GOBIN:$PATH

RUN apk --no-cache add --update \
        ruby ruby-bundler ruby-irb ruby-rdoc dnsmasq \
        procps libxslt libxml2 zlib libffi python3 go musl-dev && \
    apk --no-cache add --virtual .build-deps --update \
        ruby-dev g++ make libxslt-dev libxml2-dev zlib-dev libffi-dev && \
    wget -q -O- https://git.openstack.org/cgit/openstack/requirements/plain/upper-constraints.txt?h=$OPENSTACK_TAG | \
        sed -E s/^tempest==+\(.*\)$/-e\ git+https:\\/\\/github.com\\/openstack\\/tempest@\\1#egg=tempest/ | \
        > upper-constraints.txt && \
    git clone --depth 1 -b $VIMS_TEST_TAG https://github.com/boucherv-orange/clearwater-live-test.git /src/vims-test && \
    git clone --depth 1 -b $VIMS_TAG https://github.com/Orange-OpenSource/opnfv-cloudify-clearwater.git /src/vims && \
    git clone --depth 1 -b $VROUTER_TAG https://github.com/oolorg/opnfv-vnf-vyos-blueprint.git /src/opnfv-vnf-vyos-blueprint && \
    git clone https://github.com/RebacaInc/abot_charm.git /src/epc-requirements/abot_charm && \
    python3 -m pip install --no-cache-dir --src /src -cupper-constraints.txt \
        -chttps://git.opnfv.org/functest/plain/upper-constraints.txt?h=$BRANCH \
        juju-wait && \
    go get github.com/rogpeppe/godeps && \
    go get -d -v github.com/juju/juju/... || true && \
    (cd $GOPATH/src/github.com/juju/juju && git checkout $JUJU_TAG && godeps -u dependencies.tsv) && \
    go install -v github.com/juju/juju/... && \
    rm -rf $GOPATH/go/src/ $GOPATH/pkg && \
    (cd /src/vims-test && bundle config build.nokogiri --use-system-libraries && bundle install --system) && \
    rm -r upper-constraints.txt /src/vims-test/.git /src/vims/.git /src/opnfv-vnf-vyos-blueprint/.git \
        /src/epc-requirements/abot_charm/.git && \
    apk del .build-deps
COPY testcases.yaml /usr/lib/python2.7/site-packages/xtesting/ci/testcases.yaml
CMD ["run_tests", "-t", "all"]

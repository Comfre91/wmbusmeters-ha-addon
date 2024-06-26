ARG BUILD_FROM
FROM $BUILD_FROM AS build

ENV LANG C.UTF-8

RUN apk add --no-cache alpine-sdk gcc linux-headers librtlsdr-dev cmake libusb-dev libxml2-dev

ADD https://api.github.com/repos/comfre91/wmbusmeters/git/refs/heads/master version.json
RUN git clone https://github.com/comfre91/wmbusmeters.git && \
    git clone https://github.com/weetmuts/rtl-wmbus.git && \
    git clone https://github.com/merbanan/rtl_433.git
WORKDIR /wmbusmeters
RUN make
WORKDIR /rtl-wmbus
RUN make release && chmod 755 build/rtl_wmbus
WORKDIR /rtl_433
RUN mkdir build && cd build && cmake ../ && make

FROM $BUILD_FROM AS scratch
ENV QEMU_EXECVE=1
RUN apk add --no-cache mosquitto-clients libstdc++ curl libusb rtl-sdr netcat-openbsd nginx libxml2
# Install Flask
RUN apk add --update py3-pip py3-cryptography
RUN pip install flask requests waitress xmltodict

WORKDIR /wmbusmeters
COPY --from=build /wmbusmeters/build/wmbusmeters /wmbusmeters/wmbusmeters
COPY --from=build /rtl-wmbus/build/rtl_wmbus /usr/bin/rtl_wmbus
COPY --from=build /rtl_433/build/src/rtl_433 /usr/bin/rtl_433

ADD mqtt_discovery/ /mqtt_discovery/
COPY mqtt_discovery.sh /
RUN chmod a+x /mqtt_discovery.sh

COPY run.sh /
RUN chmod a+x /run.sh

COPY rootfs /
RUN chmod a+x /etc/services.d/nginx/run
RUN chmod a+x /etc/services.d/nginx/finish
RUN chmod a+x /etc/cont-init.d/nginx.sh

CMD ["/run.sh"]

FROM mongo:8.2.3 as build

RUN apt-get update && apt-get install -y binutils && \
    strip --strip-all /usr/bin/mongod && \
    mkdir /home/tmp

FROM alpine:3 AS sys

RUN set -xe; \
    mkdir -p /target/etc; \
    mkdir -p /blank; \
    apk --no-cache add \
      ca-certificates \
      tzdata \
    ; \
    update-ca-certificates; \
    ln -sf ../usr/share/zoneinfo/Etc/UTC /target/etc/localtime; \
    echo "Etc/UTC" > /target/etc/timezone; \
    # Create fake /sys/devices/system/cpu for tcmalloc 
    mkdir -p /target/sys/devices/system/cpu/cpu0/cache/index3; \
    mkdir -p /target/sys/devices/system/cpu/cpu0/topology; \
    mkdir -p /target/sys/devices/system/node/node0; \
    mkdir -p /target/sys/fs/cgroup; \
    echo "0" > /target/sys/devices/system/cpu/possible; \
    echo "0" > /target/sys/devices/system/cpu/online; \
    echo "0" > /target/sys/devices/system/cpu/present; \
    echo "0" > /target/sys/devices/system/node/node0/cpulist; \
    echo "0" > /target/sys/devices/system/cpu/cpu0/cache/index3/shared_cpu_list; \
    echo "0" > /target/sys/devices/system/cpu/cpu0/topology/core_cpus_list; \
    echo "0" > /target/sys/devices/system/cpu/cpu0/topology/physical_package_id; \
    echo "0" > /target/sys/devices/system/cpu/cpu0/topology/core_id; \
    echo "max" > /target/sys/fs/cgroup/memory.max; \
    # Create fake /proc files 
    mkdir -p /target/proc/sys/crypto; \
    printf "processor\t: 0\nvendor_id\t: GenuineIntel\ncpu family\t: 6\nmodel\t\t: 158\nmodel name\t: Intel Core\nstepping\t: 10\ncpu MHz\t\t: 2400.000\ncache size\t: 8192 KB\nphysical id\t: 0\nsiblings\t: 1\ncore id\t\t: 0\ncpu cores\t: 1\nflags\t\t: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc rep_good nopl xtopology cpuid pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch cpuid_fault invpcid_single ssbd ibrs ibpb stibp fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid rdseed adx smap clflushopt xsaveopt xsavec xgetbv1 xsaves arat\nbogomips\t: 4800.00\nclflush size\t: 64\ncache_alignment\t: 64\naddress sizes\t: 39 bits physical, 48 bits virtual\n\n" > /target/proc/cpuinfo; \
    echo "0" > /target/proc/sys/crypto/fips_enabled; \
    echo "1" > /target/proc/stat; \
    mkdir -p /target/etc/gnutls; \
    touch /target/etc/gnutls/config;

FROM scratch

COPY --from=sys /target/etc /etc
COPY --from=sys /usr/share/zoneinfo/Etc/UTC /usr/share/zoneinfo/Etc/UTC
COPY --from=sys /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=sys /blank /tmp
COPY --from=sys /target/sys /sys
COPY --from=sys /target/proc /proc

# OpenSSl cfg
COPY --from=build /usr/lib/ssl/openssl.cnf /usr/lib/ssl/openssl.cnf

# Binary executable
COPY --from=build /usr/bin/mongod /usr/bin/mongod

# System libraries
COPY --from=build /lib64/ld-linux-x86-64.so.2 /lib64/ld-linux-x86-64.so.2
COPY --from=build /lib/x86_64-linux-gnu/libcurl.so.4 /lib/x86_64-linux-gnu/libcurl.so.4
COPY --from=build /lib/x86_64-linux-gnu/libssl.so.3 /lib/x86_64-linux-gnu/libssl.so.3
COPY --from=build /lib/x86_64-linux-gnu/libcrypto.so.3 /lib/x86_64-linux-gnu/libcrypto.so.3
COPY --from=build /lib/x86_64-linux-gnu/libresolv.so.2 /lib/x86_64-linux-gnu/libresolv.so.2
COPY --from=build /lib/x86_64-linux-gnu/libm.so.6 /lib/x86_64-linux-gnu/libm.so.6
COPY --from=build /lib/x86_64-linux-gnu/libgcc_s.so.1 /lib/x86_64-linux-gnu/libgcc_s.so.1
COPY --from=build /lib/x86_64-linux-gnu/libc.so.6 /lib/x86_64-linux-gnu/libc.so.6
COPY --from=build /lib/x86_64-linux-gnu/libnghttp2.so.14 /lib/x86_64-linux-gnu/libnghttp2.so.14
COPY --from=build /lib/x86_64-linux-gnu/libidn2.so.0 /lib/x86_64-linux-gnu/libidn2.so.0
COPY --from=build /lib/x86_64-linux-gnu/librtmp.so.1 /lib/x86_64-linux-gnu/librtmp.so.1
COPY --from=build /lib/x86_64-linux-gnu/libssh.so.4 /lib/x86_64-linux-gnu/libssh.so.4
COPY --from=build /lib/x86_64-linux-gnu/libpsl.so.5 /lib/x86_64-linux-gnu/libpsl.so.5
COPY --from=build /lib/x86_64-linux-gnu/libgssapi_krb5.so.2 /lib/x86_64-linux-gnu/libgssapi_krb5.so.2
COPY --from=build /lib/x86_64-linux-gnu/libldap.so.2 /lib/x86_64-linux-gnu/libldap.so.2
COPY --from=build /lib/x86_64-linux-gnu/liblber.so.2 /lib/x86_64-linux-gnu/liblber.so.2
COPY --from=build /lib/x86_64-linux-gnu/libzstd.so.1 /lib/x86_64-linux-gnu/libzstd.so.1
COPY --from=build /lib/x86_64-linux-gnu/libbrotlidec.so.1 /lib/x86_64-linux-gnu/libbrotlidec.so.1
COPY --from=build /lib/x86_64-linux-gnu/libz.so.1 /lib/x86_64-linux-gnu/libz.so.1
COPY --from=build /lib/x86_64-linux-gnu/libunistring.so.5 /lib/x86_64-linux-gnu/libunistring.so.5
COPY --from=build /lib/x86_64-linux-gnu/libgnutls.so.30 /lib/x86_64-linux-gnu/libgnutls.so.30
COPY --from=build /lib/x86_64-linux-gnu/libhogweed.so.6 /lib/x86_64-linux-gnu/libhogweed.so.6
COPY --from=build /lib/x86_64-linux-gnu/libnettle.so.8 /lib/x86_64-linux-gnu/libnettle.so.8
COPY --from=build /lib/x86_64-linux-gnu/libgmp.so.10 /lib/x86_64-linux-gnu/libgmp.so.10
COPY --from=build /lib/x86_64-linux-gnu/libkrb5.so.3 /lib/x86_64-linux-gnu/libkrb5.so.3
COPY --from=build /lib/x86_64-linux-gnu/libk5crypto.so.3 /lib/x86_64-linux-gnu/libk5crypto.so.3
COPY --from=build /lib/x86_64-linux-gnu/libcom_err.so.2 /lib/x86_64-linux-gnu/libcom_err.so.2
COPY --from=build /lib/x86_64-linux-gnu/libkrb5support.so.0 /lib/x86_64-linux-gnu/libkrb5support.so.0
COPY --from=build /lib/x86_64-linux-gnu/libsasl2.so.2 /lib/x86_64-linux-gnu/libsasl2.so.2
COPY --from=build /lib/x86_64-linux-gnu/libbrotlicommon.so.1 /lib/x86_64-linux-gnu/libbrotlicommon.so.1
COPY --from=build /lib/x86_64-linux-gnu/libp11-kit.so.0 /lib/x86_64-linux-gnu/libp11-kit.so.0
COPY --from=build /lib/x86_64-linux-gnu/libtasn1.so.6 /lib/x86_64-linux-gnu/libtasn1.so.6
COPY --from=build /lib/x86_64-linux-gnu/libkeyutils.so.1 /lib/x86_64-linux-gnu/libkeyutils.so.1
COPY --from=build /lib/x86_64-linux-gnu/libffi.so.8 /lib/x86_64-linux-gnu/libffi.so.8

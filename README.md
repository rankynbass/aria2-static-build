# aria2-static-build

aria2 static build using musl and support many platforms.

## x86_64 builds for use with XIVLauncher.Core

Out of an abundance of caution, I've forked this repo and modified the build script. It now runs with a normal ubuntu:24.04 container, instead of pulling a custom docker container.

I haven't got the continuous integration or Github actions working yet, but hopefully I'll get that working soon.

The 1.37.0 release was built in an Arch virtual machine with the included script.

Everything below this line was taken from the original repo, modified slightly to make more sense for this use case.

## Build locally yourself

Requirements:

- docker or podman

```sh
docker run --rm -v `pwd`:/build docker.io/library/ubuntu:24.04 /build/build.sh
```

If you want to build for other platform, you may have to modify `build.sh` to suitable for your platform.

Cached build dependencies (`downloads/`), `build_info.md` and `aria2c` will be found in current directory.

You can set more optional environment variables in `docker` command like:

```sh
docker run --rm -v `pwd`:/build -e USE_ZLIB_NG=0 -e USE_LIBRESSL=1 docker.io/library/ubuntu:24:04 /build/build.sh
```

Optional environment variables:

- `ARIA2_VER`: build specific version of aria2, e.g: `1.36.0`. Default: `master`.
- `USE_ZLIB_NG`: use [zlib-ng](https://github.com/zlib-ng/zlib-ng) instead of [zlib](https://zlib.net/). Default: `1`
- `USE_LIBRESSL`: use [LibreSSL](https://www.libressl.org/) instead of [OpenSSL](https://www.openssl.org/). Default: `0`. **_NOTE_**, if `CROSS_HOST=x86_64-w64-mingw32` will not use openssl or libressl because aria2 and all dependencies will use WinTLS instead.

## https certificates NOTE (Linux Only)

SSL certificates location may vary from different distributions. E.g: Ubuntu uses `/etc/ssl/certs/ca-certificates.crt`, but CentOS uses `/etc/pki/tls/certs/ca-bundle.crt`.

It's impossible to detect certificates location in all distributions. See issue: [openssl/openssl#7481](https://github.com/openssl/openssl/issues/7481). But luckily most distributions may contains a symbol link `/etc/ssl/cert.pem` which point to the actual file path.

So I set compile options `--openssldir=/etc/ssl/` for openssl/libressl. Which works for most distributions.

If your environment contains file `/etc/ssl/openssl.cnf` or `/etc/ssl/cert.pem`, you were luckly and you can use my build out-of-box.

But if your environment does not contain any of the files, you have to do one of the following settings to make https request could work.

- add `--ca-certificate=/path/to/your/certificate` to `aria2c` or set `ca-certificate=/path/to/your/certificate` in `~/.aria2/aria2.conf`. E.g: `./aria2c --ca-certificate=/etc/pki/tls/certs/ca-bundle.crt https://github.com/`
- Or add `SSL_CERT_FILE=/path/to/your/certificate` environment variable before you run `aria2c`. E.g: `export SSL_CERT_FILE=/etc/pki/tls/certs/ca-bundle.crt; ./aria2c https://github.com/` or `SSL_CERT_FILE=/etc/pki/tls/certs/ca-bundle.crt ./aria2c https://github.com/`

> Reference for different distribution certificates locations: https://gitlab.com/probono/platformissues/blob/master/README.md#certificates
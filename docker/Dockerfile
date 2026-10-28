# Bun and Node come from pinned official images (glibc, like the bookworm PHP bases).
FROM oven/bun:1.3.12-debian@sha256:1b709c9dd883fc1af38c210f7ea5222c552a8d470ea73efbd4b8fcfee798a64b AS bun
FROM node:24-bookworm-slim@sha256:55fb5f5a527ea2c5c4a95f963e48ef90aa38c7b346054fdc8ee9957a2b55d205 AS node

FROM php:8.4-cli-bookworm AS dev
RUN apt-get update && apt-get install -y --no-install-recommends \
    git curl ca-certificates libcurl4-openssl-dev unzip \
    && docker-php-ext-install pcntl curl \
    && rm -rf /var/lib/apt/lists/*
COPY --from=bun /usr/local/bin/bun /usr/local/bin/bun
COPY --from=node /usr/local/bin/node /usr/local/bin/node
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
RUN ln -s bun /usr/local/bin/bunx \
    && ln -s ../lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm \
    && ln -s ../lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx \
    && ln -s ../lib/node_modules/corepack/dist/corepack.js /usr/local/bin/corepack
RUN curl -fsSL -o /usr/local/bin/php-cs-fixer https://github.com/PHP-CS-Fixer/PHP-CS-Fixer/releases/download/v3.95.25/php-cs-fixer.phar \
    && echo "80cad475fc5112fdbfab8bd66e51665ed78c1b849b918dab81fb63b7a7003b41  /usr/local/bin/php-cs-fixer" | sha256sum -c - \
    && chmod +x /usr/local/bin/php-cs-fixer
WORKDIR /app

FROM php:8.4-fpm-bookworm AS fpm
RUN apt-get update && apt-get install -y --no-install-recommends \
    git curl ca-certificates libcurl4-openssl-dev unzip \
    && docker-php-ext-install pcntl curl \
    && rm -rf /var/lib/apt/lists/*
COPY --from=bun /usr/local/bin/bun /usr/local/bin/bun
COPY --from=node /usr/local/bin/node /usr/local/bin/node
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
RUN ln -s bun /usr/local/bin/bunx \
    && ln -s ../lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm \
    && ln -s ../lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx \
    && ln -s ../lib/node_modules/corepack/dist/corepack.js /usr/local/bin/corepack
COPY docker/php/fpm.conf /usr/local/etc/php-fpm.d/zz-jit.conf
WORKDIR /app
RUN mkdir -p /var/jit-state && chown www-data:www-data /var/jit-state
CMD ["php-fpm", "-F"]

FROM dev AS cli
CMD ["php", "supervisor/cli.php"]

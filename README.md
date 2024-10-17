# rust-build-scripts
Инструкция и скрипты для сборки `Rust` проектов. Предназначены в первую очередь для тех, кто хочет собрать мои проекты, но c `Rust` не знаком.

# Сборка вручную
Для начала необходимо установить `Rust`, см. инструкцию на [оф. сайте](https://www.rust-lang.org/tools/install).  
Если планируется запускать на современных процессорах, то рекомендую выполнять следующую команду:
```bash
RUSTFLAGS='-C target-cpu=x86-64-v3' cargo build --release
```
или аналогично для powershell:
```powershell
$env:RUSTFLAGS="-C target-cpu=x86-64-v3"; cargo build --release
```
для `msvc` рекомендую статически линковать CRT(C runtime):
```powershell
$env:RUSTFLAGS="-C target-cpu=x86-64-v3 -C target-feature=+crt-static"; cargo build --release
```

Если планируется запускать на этом же компьютере, то тогда можно изменить таргет на: `target-cpu=native`. Если же планируется запускать на старом процессоре, то: `target-cpu=x86-64-v2`, либо можно выбрать конкретную архитектуру: `target-cpu=skylake`. Список всех таргетов см. с помощью команды: `rustc --print target-cpus`.

При сборке на линукс нужно иметь ввиду, что собранная программа не сможет запуститься на более старой ОС, а точнее с более старой версией glibc (проверить можно командой `ldd --version`). В таком случае, лучше выполнить сборку в контейнере, см. инструкцию дальше.

# Сборка в контейнере
Главные преимущества:
- Не требуется установка `Rust`(но требуется установленный `docker` или `podman`).
- На `linux` можно собрать бинарники для `windows`.

Я рекомендую использовать `podman`, но если вы предпочитаете `docker`, то просто замените одно на другое (их параметры совместимы). Также, возможно, потребуется изменить `target-cpu`, о том, как выбрать нужное значение, было рассказано в предыдущем разделе.

Сборка для Linux:
```bash
podman run -it --rm \
    --volume $PWD:/project \
    --env RUSTFLAGS='-C target-cpu=x86-64-v3' \
    docker.io/library/ubuntu:18.04 sh -c \
    "apt-get -y update ;\
    apt-get -y install wget build-essential lld ;\
    wget --no-verbose https://static.rust-lang.org/rustup/dist/x86_64-unknown-linux-gnu/rustup-init ;\
    chmod +x rustup-init ;\
    ./rustup-init -y ;\
    /root/.cargo/bin/cargo build --manifest-path /project/Cargo.toml --release"
```

Сборка для Windows:
```bash
podman run -it --rm \
    --volume $PWD:/project \
    --env RUSTFLAGS='-C target-cpu=x86-64-v3' \
    docker.io/library/rust:1-bullseye sh -c \
    "apt-get -y update ;\
    apt-get -y install mingw-w64 ;\
    rustup target add x86_64-pc-windows-gnu ;\
    cargo build --manifest-path /project/Cargo.toml --release --target x86_64-pc-windows-gnu"
```

# Rust Driver

- **Alunes:** Matheus Kwon / Paulo Souza Chade
- **Curso:** Engenharia da Computação
- **Semestre:** 6
- **Contato:** kwonmat@outlook.com.br / paulosc4@al.insper.edu.br
- **Ano:** 2022

!!! info 
    [Repositório original](https://github.com/pauloschade/linux-rust)

## Começando

Para seguir esse tutorial é necessário:

- **Hardware:** -
- **Softwares:** C-Lang versão >= 11, GLIBC 2.32 (Uma solução fácil para utilizar esta versão do GLIBC é utilizar o [multipass](https://multipass.run/))
- **Repositórios:** [Base Linux-Rust mínima](https://github.com/pauloschade/linux-rust), [JackOS](https://github.com/jackos/linux) e [Linux](https://github.com/Rust-for-Linux/linux)

## Motivação

Para a matéria de SoC - Linux Embarcado, nós observamos uma oportunidade de realizar um projeto final no qual poderíamos aprofundar no escopo de módulos e drivers Linux. Visando aprender mais, optamos por criar drivers na linguagem Rust. Portanto, este tutorial teve como principal motivação o aprendizado em Rust.

----------------------------------------------

## Repositório

O repositório utilizado é um adaptado por nós para este tutorial:

```
https://github.com/pauloschade/linux-rust
```

Não é necessário cloná-lo agora, pois já iremos fazer isso em instantes. Ele é um derivado distante do repositório do [Linux original](https://github.com/torvalds/linux), com modificações para que suporte módulos em Rust e mais simplificado. Ele não possui algumas *features* comuns, como *make*, *gcc*, *vim*, *ssh* etc., mas possui essenciais para o funcionamento do *driver*.

## Dependências utilizadas

Para instalar as dependências que serão utilizadas na aplicação, você deverá executar os comandos abaixo no terminal:

Instalação de pacotes necessários:
```sh
sudo apt update
sudo apt install -y bc bison curl clang fish flex git gcc libclang-dev libelf-dev lld llvm-dev libncurses-dev make neovim qemu-system-x86
```

Exportar essas variáveis:
```sh
export PATH="/root/.cargo/bin:${PATH}"
export MAKEFLAGS="-j16"
export LLVM="1"
```

Agora, inicie uma nova aba ou janela do terminal.

Instalação de rustup:
```sh
curl https://sh.rustup.rs -sSf | bash -s -- -y
```

Instalação da versão necessária do bindgen:
```sh
git clone https://github.com/rust-lang/rust-bindgen -b v0.56.0 --depth=1
cargo install --path rust-bindgen
```

Clone do repositório de Base mínima de Linux-Rust:
```sh
git clone https://github.com/pauloschade/linux-rust --depth=1
cd linux
```

Configurando a versão de rustc:
```sh
rustup override set $(scripts/min-tool-version.sh rustc)
rustup component add rust-src
```

Build inicial para testar se tudo até agora está certo:
```sh
make allnoconfig qemu-busybox-min.config rust.config
make
```

## Realizando build da máquina virtual Linux

Para testar o funcionamento da máquina virtual, rode:
```
make rustvm
```

Se tudo ocorrer bem, você verá o seguinte print no log de boot:

```
--------------------------------------
--------------------------------------
--------------------------------------
Hello world from rust!
--------------------------------------
--------------------------------------
--------------------------------------
```

Isso acontecerá, pois há um módulo criado que se chama "HelloWorld", que executa a função de realizar display dessas linhas. Ele está definido no arquivo *samples/rust/rust_hello.rs* do repositório.

## Identificando um módulo

Iniciando a criação do *driver* em Rust, crie um arquivo no diretório *samples/rust/* com o nome "rust_ebbchar.rs". Vamos preencher daqui a pouco este arquivo, mas, por enquanto pode deixar ele vazio.

Então, no mesmo diretório, existem os arquivos *Kconfig* e *Makefile*, que são responsáveis por identificar módulos em Rust adicionados. No arquivo *Kconfig*, existe o seguinte bloco de código:
```
config SAMPLE_RUST_HELLO
	tristate "Hello"
	help
	  This option builds the Rust Hello module.

	  If unsure, say N.
```
o qual está envolvido por um bloco de "if". Dentro do bloco "if", adicione o seguinte bloco semelhante ao acima e depois dele:
```
config SAMPLE_RUST_EBBCHAR
	tristate "Ebbchar"
	help
	  This option builds the Rust Ebbchar module.

	  If unsure, say N.
```
No arquivo *Makefile*, existe apenas a seguinte linha de código:
```
obj-$(CONFIG_SAMPLE_RUST_HELLO)		+= rust_hello.o
```
Abaixo dela, adicione:
```
obj-$(CONFIG_SAMPLE_RUST_EBBCHAR)	+= rust_ebbchar.o
```

Assim, temos a infraestrutura para identificar o driver que adicionaremos em "rust_ebbchar.rs".

## Criando um módulo

Agora, no arquivo "rust_ebbchar.rs", vamos ter os seguintes blocos de código:
- Imports
- Definições do módulo e structs
- Implementação do Device
- Implementação do módulo

O bloco de imports:
```
use kernel::prelude::*;
use kernel::io_buffer::{IoBufferReader, IoBufferWriter};
use kernel::prelude::*;
use kernel::sync::{smutex::Mutex, Ref, RefBorrow};
use kernel::{file, miscdev};
```

Definindo o módulo:
```
module! {
    type: Ebbchar,
    name: b"ebbchar",
    description: b"A simple ebbchar example",
    license: b"GPL",
}
```

Agora, definindo as *structs*:
```
struct Ebbchar {
    _dev : Pin<Box<miscdev::Registration<Ebbchar>>>,
}

struct Device {
    number: usize,
    contents: Mutex<Vec<u8>>,
}
```

Por padrão uma variavél de rust só dura dentro do escopo que foi criada, dessa forma é necessária uma estrutura para armazenar o registro do driver (caso contrário ele seria destruida logo após a funcão ```init```).

Para isso que foi adicionado o seguinte atributo à struct Ebbchar

```
 _dev : Pin<Box<miscdev::Registration<Ebbchar>>>,
```

!!! note
    O Box do permite que você armazene dados no Heap ao invés da memória

!!! note
    O Pin serve como uma garantia que o endereco da variável é estatico, ou seja, é impossível de trocar até a variável ser destruída.

    Ele é muito importante pois em rust, por padrão, os enderecos são mutáveis. Ou seja, se passarmos a struct Ebbchar para uma funcão, ela
    e seu conteúdo, podem ser copiados para outro endereco (por causa do compilador de rust, é como se automaticamente chamasse memcpy), mas mesmo assim o ponteiro Box estará apontando pro endereco antigo de Ebbchar (porque possui uma Self-reference), portanto neste caso ele estaria apontando para algum endereco inválido.

A implementação do Ebbchar:
```
#[vtable]
impl file::Operations for Ebbchar {
    type OpenData = Ref<Device>;
    type Data = Ref<Device>;

    fn open(context: &Ref<Device>, file: &file::File) -> Result<Ref<Device>> {
        pr_info!("File for device {} was opened\n", context.number);
        if file.flags() & file::flags::O_ACCMODE == file::flags::O_WRONLY {
            context.contents.lock().clear();
        }
        Ok(context.clone())
    }

    fn read(
        data: RefBorrow<'_, Device>,
        _file: &file::File,
        writer: &mut impl IoBufferWriter,
        offset: u64,
    ) -> Result<usize> {
        pr_info!("File for device {} was read\n", data.number);
        let offset = offset.try_into()?;
        let vec = data.contents.lock();
        let len = core::cmp::min(writer.len(), vec.len().saturating_sub(offset));
        writer.write_slice(&vec[offset..][..len])?;
        Ok(len)
    }

    fn write(
        data: RefBorrow<'_, Device>,
        _file: &file::File,
        reader: &mut impl IoBufferReader,
        offset: u64,
    ) -> Result<usize> {
        pr_info!("File for device {} was written\n", data.number);
        let offset = offset.try_into()?;
        let len = reader.len();
        let new_len = len.checked_add(offset).ok_or(EINVAL)?;
        let mut vec = data.contents.lock();
        if new_len > vec.len() {
            vec.try_resize(new_len, 0)?;
        }
        reader.read_slice(&mut vec[offset..][..len])?;
        Ok(len)
    }
}
```

Por fim, a implementação do módulo:
```
impl kernel::Module for Ebbchar {
    fn init(_name: &'static CStr, _module: &'static ThisModule) -> Result<Self> {
        pr_info!("--------------------------------------\n");
        pr_info!("--------------------------------------\n");
        pr_info!("--------------------------------------\n");
        pr_info!("Ebbchar initialized!\n");
        pr_info!("--------------------------------------\n");
        pr_info!("--------------------------------------\n");
        pr_info!("--------------------------------------\n");
        let dev = Ref::try_new(Device {
            number: 0,
            contents: Mutex::new(Vec::new()),
        })?;
        let reg = miscdev::Registration::new_pinned(fmt!("ebbchar"), dev)?;
        Ok(Self { _dev: reg })
    }
}
```

A linha seguinte:

```
let reg = miscdev::Registration::new_pinned(fmt!("ebbchar"), dev)?;
```
registra o modulo como um driver. 

!!! note 
    Sobre o ```?``` ao final da linha
    Ele é um tratamento de erro. Caso o registro falhe é avisado ao usuário que deu errado (no run time). Caso contrário, é alocado para variável reg.

    equivalente a fazer o código a seguir.
    ```
    if let Err(e) = reg {
        //Do something
    }
    ```

## Explicando as funcões do driver

- ```open```: chamada quando o driver for aberto
    - Como ```dev_open``` em C
    - Recebe argumentos *context:OpenData* e *_file:&file::File*. Retorna um novo objeto do tipo *Ref<Device>*
    - Tipo do argumento *OpenData* pode ser mudado de acordo com necessidade. Em nosso caso: ```type OpenData = Ref<Device>;```. 
    - ```Ref``` é uma forma de fazer *reference counting* de uma struct Device. No rust padrão ```Ref``` é chamado de ```Arc```.
    - Esta é a forma de fazer com que duas variáveis do tipo ```Device``` apontem para o mesmo endereco no Heap (afinal apenas temos um *único* Device que foi criado)
    - ```.clone``` retorna uma variável do tipo ```Device``` referenced counted

- ```read```: chamada quando o driver for lido
    - Como ```dev_read``` em C
    - Recebe argumentos *data: Data*, *_file:&file::File*, *writer:&mut impl IoBufferWriter* e *offset: u64*
    - Tipo do argumento *Data* pode ser mudado de acordo com necessidade. Em nosso caso: ```type Data = Ref<Device>;```. 
    - ```RefBorrow<T>```significa que vamos receber uma varíavel *referenced counted* do tipo *T*
    - Printa a mensagem guardada em ```Device::contents```
    - *obs*: o "_" é utilizado em rust para sinalizar que a variável não será utilizada (ex: *_file*)
  
- ```write```: chamada quando o driver for escrito
    - Como ```dev_write``` em C
    - Recebe argumentos *data: Data*, *_file:&file::File*, *reader:&mut impl IoBufferReader* e *offset: u64*
    - Muda o ```Device::contents``` da struct Device

!!! note
    É necessário ter um *Mutex* pois estamos trabalhando com várias variáveis Device com o mesmo endereco (referenced counted). Assim evitamos ter mais de um acesso a ```Device::contents``` ao mesmo tempo.

## Rodando o Linux-Rust

Para que a inicialização do Linux virtual funcione, baixe os arquivos *[.config](https://github.com/matkwon/RustDriver/raw/master/files/.config)* e *[Makefile](https://github.com/pauloschade/linux-rust/raw/rust/Makefile)* no diretório raiz do repositório clonado.

Por fim, execute a linha de comando:
```
make rustvm
```

A máquina virtual irá se iniciar com o *driver* em Rust. Se tudo correr bem, no terminal, será mostrada uma saída assim:

```
--------------------------------------
--------------------------------------
--------------------------------------
Ebbchar initialized!
--------------------------------------
--------------------------------------
--------------------------------------
```

Mas, para testar se está realmente funcionando, execute ```echo "funciona!" > /dev/ebbchar```, que é um comando que escreve no dispositivo do *driver*. Caso esteja tudo certo, será impresso algo parecido com:

```
ebbchar: File for device 0 was opened
ebbchar: File for device 0 was written
```

Para ler o que foi escrito no dispositivo, rode ```cat /dev/ebbchar```, que imprimirá algo como:

```
ebbchar: File for device 0 was opened
ebbchar: File for device 0 was read
funciona!
ebbchar: File for device 0 was read
```

Caso queira testar mais a fundo o *driver*, baixe na máquina virtual um executável que foi compilado de um código em C:

```
wget https://github.com/pauloschade/linux-rust/raw/complete/rust_driver_comm/main
```

Este executável pede para o usuário realizar um *input* que será escrito no driver. Então, ele irá ler o *driver* e reproduzir o que acabou de ser enviado ao dispositivo. Para rodar o executável, basta rodar ```./main```.

Caso queira entender o código em C gerador deste executável, visite [aqui](https://github.com/pauloschade/linux-rust/blob/complete/rust_driver_comm/main.c).
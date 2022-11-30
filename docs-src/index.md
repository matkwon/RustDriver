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

Isso acontecerá, pois há um módulo criado que se chama "HelloWorld", que executa a função de realizar display dessas linhas. Ele está definido no arquivo "samples/rust/rust_hello.rs" do repositório.

## 

Iniciando a criação do *driver* em Rust, crie um arquivo no diretório "samples/rust/" com o nome "rust_ebbchar.rs".

Para que a inicialização do Linux virtual funcione, baixe [este](https://github.com/matkwon/RustDriver/raw/master/files/.config) arquivo no diretório raiz do repositório clonado.

## Recursos Markdown

Vocês podem usar tudo que já sabem de markdown mais alguns recursos:

!!! note 
    Bloco de destaque de texto, pode ser:
    
    - note, example, warning, info, tip, danger
    
!!! example "Faça assim"
    É possível editar o título desses blocos
    
    !!! warning
        Isso também é possível de ser feito, mas
        use com parcimonia.
    
??? info 
    Também da para esconder o texto, usar para coisas
    muito grandes, ou exemplos de códigos.
    
    ```txt
    ...
    
    
    
    
    
    
    
    
    
    
    
    oi!
    ```
    
- **Esse é um texto em destaque**
- ==Pode fazer isso também==

Usar emojis da lista:

:two_hearts: - https://github.com/caiyongji/emoji-list


```c
// da para colocar códigos
 void main (void) {}
```

É legal usar abas para coisas desse tipo:
    
=== "C"

    ``` c
    #include <stdio.h>

    int main(void) {
      printf("Hello world!\n");
      return 0;
    }
    ```

=== "C++"

    ``` c++
    #include <iostream>

    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ```

Inserir vídeo:

-  Abra o youtube :arrow_right: clique com botão direito no vídeo :arrow_right: copia código de incorporação:

<iframe width="630" height="450" src="https://www.youtube.com/embed/UIGsSLCoIhM" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

!!! tip
    Eu ajusto o tamanho do vídeo `width`/`height` para não ficar gigante na página
    
Imagens você insere como em plain markdown, mas tem a vantagem de poder mudar as dimensões com o marcador `{width=...}`
    
![](icon-elementos.png)

![](icon-elementos.png){width=200}

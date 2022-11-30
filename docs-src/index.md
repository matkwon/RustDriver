# Rust Driver

- **Alunes:** Matheus Kwon / Paulo Souza Chade
- **Curso:** Engenharia da Computação
- **Semestre:** 6
- **Contato:** kwonmat@outlook.com.br / paulosc4@al.insper.edu.br / corsiferrao@gmail.com
- **Ano:** 2022

!!! info 
    [Repositório original](https://github.com/pauloschade/linux-rust)

## Começando

Para seguir esse tutorial é necessário:

- **Hardware:** Ubuntu 22.04 (Jammy Jellyfish)
- **Softwares:** -
- **Repositórios:** [JackOS](https://github.com/jackos/linux) e [Linux](https://github.com/Rust-for-Linux/linux)

## Motivação

Para a matéria de SoC - Linux Embarcado, nós observamos uma oportunidade de realizar um projeto final no qual poderíamos aprofundar no escopo de módulos e drivers Linux. Visando aprender mais, optamos por criar drivers na linguagem Rust. Portanto, este tutorial teve como principal motivação o aprendizado em Rust.

----------------------------------------------

## Realizando build da máquina virtual Linux



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

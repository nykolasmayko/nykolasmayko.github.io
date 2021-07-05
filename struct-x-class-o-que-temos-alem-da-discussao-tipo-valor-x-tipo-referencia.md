# Struct x Class, o que temos além da discussão Tipo Valor x Tipo Referência?

Certamente, você já deve ter se perguntado a diferença entre _Struct_ e _Class_ enquanto você escrevia seu código, não é mesmo? Por que _models_ são _Struct_ e _ViewControllers_ são _Class?_ Provavelmente, isso já passou em algum momento na sua cabeça, então, numa rápida pesquisa, no _Google,_ sobre as diferenças entre _Struct_ e _Class_ em _Swift,_ você deve ter encontrado que um é **Tipo Valor** e a outra **Tipo Referência**. Mas, o que isso quer dizer? O que mais podemos descobrir se mergulharmos uma pouco mais fundo nessa “simples" diferença?

Então, vem comigo que vou tentar explicar um pouco ~esse assunto daria uma tese de Doutorado~ do que eu encontrei e achei bastante interessante compartilhar. Em resumo, seguiremos os seguintes tópicos:

- O que são _Struct_ e _Class?_
- O que significa, especificamente, **Tipo Valor** e **Tipo Referência**?
	- _Stack_ x _Heap_
- Gerenciamento de Memória
	- Contador Automático de Referências (_ARC)_
	- _Strong_ x _Weak_
	- Ciclos de Referências Fortes


### 1) **O que são _Struct_ e _Class?_**

_Struct_ e _Class_ são estruturas flexíveis de uso geral que, juntas, se tornam como blocos de construção, dentro do código do seu programa. Ou seja, você define propriedades e funções para adicionar funcionalidades a suas _Structs_ e _Classes_ usando a mesma sintaxe que você usa para definir constantes, variáveis e funções. Todo e qualquer tipo abstrato de dado que você criará para realizar suas abstrações poderão ser do tipo _Struct_ ou _Class,_ onde lá estarão todas as propriedades e comportamentos da abstração desenvolvida.

**_Structs_** **e _Classes_, em Swift, possuem muitas coisas em comum. Ambas podem:**
-   Definir propriedades para armazenar valores;
-   Definir métodos que fornecem funcionalidades;
-   Definir _subscripts_ para fornecer acesso aos seus valores usando a sintaxe _subscript_;
-   Definir inicializadores para configurar seus estados iniciais;
-   Serem extendidas para expandir suas funcionalidades, além da implementação inicial;
-   Implementar protocolos (_protocols)_ para fornecer funcionalidades padrão a um certo tipo.

_Classes_ possuem capacidades adicionais que _Structs_ não possuem:

-   A Herança permite que uma classe herde características de uma outra classe;
-   Conversão de tipo permite que você cheque e interprete o tipo de uma instancia de uma classe em tempo de execução;
-   Deinicializadores permitem que a instancia de uma classe liberar a memória de quaisquer recursos a ela atribuídos;

Portanto, percebe-se que _Struct_ e _Classes_ possuem bastante coisas em comum, mas possuem **diferenças chaves** que nos fazem ter que pensar no uso mais adequado para a implementação que será realizada. De acordo com [_https://developer.apple.com/documentation/swift/choosing_between_structures_and_classes_](https://developer.apple.com/documentation/swift/choosing_between_structures_and_classes) temos alguns pontos que podem nos ajudar a decidir entre a utilização de uma _Struct_ ou _Class_.

### **2) O que significa, especificamente, Tipo Valor e Tipo Referência?**

Um **Tipo Valor** é um tipo no qual o **valor** é **copiado** quando é atribuído a uma variável ou constante, ou quando é passado por uma função. Todos os tipos básicos em Swift: _integers, floating-point numbers, Booleans, strings, arrays_ e _dictionaries_ são tipo valor e são implementados como _structures_ por “baixo dos panos”.

Todas **_structures_ e _enumerations_ são tipo valor**. Isso significa que qualquer instância de uma _struct_ ou _enumeration_ que você crie (e qualquer propriedade **tipo valor** que elas possuam) serão copiados quando forem passados em seu código.

Abaixo, temos um exemplo de como se comporta a atribuição de uma _Struct_ e a modificação de suas propriedades:
```swift
struct Resolution {
var width = 0
var height = 0
}

var hd = Resolution(width: 1920, height: 1080)
var cinema = hd
cinema.width = 2048

print("A largura do Cinema agora é de \(cinema.width) pixels")
// A saída será "A lagura do cinema é 2048 pixels"
```

Já o  **Tipo Referência não é copiado** quando atribuídos a uma variável ou constante, ou passados para uma função. Ao invés de uma cópia, uma referência com a mesma instância existente é usada. 

> Se você possui experiência com C, C++ ou Objective-C, você deve saber que essas linguagens usam ponteiros para se referir ao endereço de memória, em Swift, uma constante ou variável que “apontam" para uma instância de algum tipo referência é **similar** a um ponteiro em C, mas **não é um ponteiro** para um endereço de memória diretamente e não requer que você escreva um asterisco para indicar que você está criando uma referência.

Abaixo, tem-se um exemplo do comportamento quando atribuímos uma propriedade Tipo Referência a uma variável ou constante.

```swift
class VideoMode {
	var resolution = Resolution()
	var interlaced = false
	var frameRate = 0.0
	var name: String?
}

let tenEighty = VideoMode()
tenEighty.resolution = hd
tenEighty.interlaced = true
tenEighty.name = "1080i"
tenEighty.frameRate = 25.0

let alsoTenEighty = tenEighty
alsoTenEighty.frameRate = 30.0

print("A propriedade frameRate da constante tenEighty agora é \(tenEighty.frameRate)")
// A saída será "A propriedade frameRate da constante tenEighty agora é 30.0"
```



#### 2.1 _Stack_ x _Heap_

Agora que entendemos a diferença entre variáveis ou constantes Tipo Valor e Tipo Referência, vamos mergulhar um pouco mais sobre profundo e entender onde cada uma das propriedades são guardadas na memória quando são do Tipo Valor e quando são Tipo Referência.

**Memória** é apenas uma longa lista de bytes. Os bytes são organizados ordenadamente, todo byte tem seu próprio endereço. Um intervalo discreto de endereços é conhecido como um **espaço de endereço** ou ***address space***.

O *address space* de uma aplicação iOS, logicamente, consiste em 4 segmentos: texto, dados, a ***stack*** e o ***heap***:

![](https://www.vadimbulavin.com/assets/images/posts/value-reference-types/memory-segments.svg)
*Fonte: https://www.vadimbulavin.com/value-types-and-reference-types-in-swift/*

Em linhas gerais, abaixo, pode-se verificar as responsabilidades de cada uma das seções:

- O segmento de **Texto** contém a máquina de instruções que forma o código executável do app. O segmento é *read-only* e ocupa um espaço constante;
- O segmento de **Dados**  armazena as variáveis estáticas do Swift, constantes, etc. Todos os dados globais que necessitam de um valor inicial quando o programa é iniciado.
- A ***Stack*** armazena dados temporários: parâmetro de funções e variáveis locais. Sempre que chamamos uma função, uma nova peça de memória é alocada na *stack*. Essa memória é liberada quando o método é encerrado.
- O ***Heap*** armazena os objetos que possuem um tempo de vida. Todos são Tipo Refência e, alguns casos, Tipo Valor. O *heap* e a *stack* crescem em direção à outra.

 

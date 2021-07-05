# Struct x Class, o que temos além da discussão Tipo Valor x Tipo Referência?

Certamente, você já deve ter se perguntado a diferença entre _Struct_ e _Class_ enquanto você escrevia seu código, não é mesmo? Por que _models_ são _Struct_ e _ViewControllers_ são _Class?_ Provavelmente, isso já passou em algum momento na sua cabeça, então, numa rápida pesquisa, no _Google,_ sobre as diferenças entre _Struct_ e _Class_ em _Swift,_ você deve ter encontrado que um é **Tipo Valor** e a outra **Tipo Referência**. Mas, o que isso quer dizer? O que mais podemos descobrir se mergulharmos uma pouco mais fundo nessa “simples" diferença?

Então, vem comigo que vou tentar explicar um pouco ~esse assunto daria uma tese de Doutorado~ do que eu encontrei e achei bastante interessante compartilhar. Em resumo, seguiremos os seguintes tópicos:

- O que são _Struct_ e _Class?_
- O que significa, especificamente, **Tipo Valor** e **Tipo Referência**?
	- _Stack_ x _Heap_
- Gerenciamento de Memória
	- Contador Automático de Referências (_ARC)_
		- _ARC_ em ação
	- _Weak_ x _Unowned_
	- Ciclos de Referências Fortes
- Referências

## 1) **O que são _Structures_ e _Class?_**

_Struct_ e _Class_ são estruturas flexíveis de uso geral que, juntas, se tornam como blocos de construção dentro do código do seu programa. Ou seja, você define propriedades e funções para adicionar funcionalidades a suas _Structures_ e _Classes_ usando a mesma sintaxe que você usa para definir constantes, variáveis e funções. Todo e qualquer tipo abstrato de dado que você criará, para realizar suas abstrações, poderão ser do tipo _Struct_ ou _Class_, lá estarão todas as propriedades e comportamentos da abstração desenvolvida.

**_Structures_** **e _Classes_, em Swift, possuem muitas coisas em comum. Ambas podem:**
-   Definir propriedades para armazenar valores;
-   Definir métodos que fornecem funcionalidades;
-   Definir _subscripts_ para fornecer acesso aos seus valores usando a sintaxe _subscript_;
-   Definir inicializadores para configurar seus estados iniciais;
-   Serem extendidas para expandir suas funcionalidades, além da implementação inicial;
-   Implementar protocolos (_protocols)_ para fornecer funcionalidades padrão a um certo tipo.

_Classes_ possuem capacidades adicionais que _Structures_ não possuem:

-   A Herança permite que uma classe herde características de uma outra classe;
-   Conversão de tipo permite que você cheque e interprete o tipo de uma instancia de uma classe em tempo de execução;
-   Deinicializadores permitem que a instancia de uma classe liberar a memória de quaisquer recursos a ela atribuídos;

Portanto, percebe-se que _Struct_ e _Classes_ possuem bastante coisas em comum, mas possuem **diferenças chaves** que nos fazem ter que pensar no uso mais adequado para a implementação que será realizada. De acordo com [Choosing Between Structures and Classes](https://developer.apple.com/documentation/swift/choosing_between_structures_and_classes) temos alguns pontos que podem nos ajudar a decidir entre a utilização de uma _Struct_ ou _Class_.

## **2) O que significa, especificamente, Tipo Valor e Tipo Referência?**

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

Agora que entendemos a diferença entre variáveis ou constantes Tipo Valor e Tipo Referência, vamos mergulhar um pouco mais profundo e entender onde cada uma das propriedades são guardadas na memória quando são do Tipo Valor e quando são Tipo Referência.

**Memória** é apenas uma longa lista de bytes. Os bytes são organizados ordenadamente e todo byte tem seu próprio endereço. Um intervalo discreto de endereços é conhecido como um **espaço de endereço** ou ***address space***.

O *address space* de uma aplicação iOS, logicamente, consiste em 4 segmentos: texto, dados, a ***stack*** e o ***heap***:

![](https://www.vadimbulavin.com/assets/images/posts/value-reference-types/memory-segments.svg)
Fonte: [https://www.vadimbulavin.com/value-types-and-reference-types-in-swift/](https://www.vadimbulavin.com/value-types-and-reference-types-in-swift/)

Em linhas gerais, abaixo, define-se as responsabilidades de cada uma das seções:

- O segmento de **Texto** contém a máquina de instruções que forma o código executável do app. O segmento é *read-only* e ocupa um espaço constante;
- O segmento de **Dados**  armazena as variáveis estáticas do Swift, constantes, etc. Todos os dados globais que necessitam de um valor inicial quando o programa é iniciado;
- A ***Stack*** armazena dados temporários: parâmetro de funções e variáveis locais. Sempre que chamamos uma função, uma nova peça de memória é alocada na *stack*. Essa memória é liberada quando o método/função é encerrado;
- O ***Heap*** armazena os objetos que possuem um tempo de vida. Todos são Tipo Refência e, alguns casos, Tipo Valor. O *heap* e a *stack* crescem em direção à outra.

Considere o código abaixo:
```swift
class VideoMode {}
struct Resolution {}
enum ResolutionType {}
protocol ResolutionDelegate {}
```

A alocação, na memória, seguirá a tabela:

| Stack | Heap |
|--------|------|
|Resolution | VideoMode |
|ResolutionType | \<Class\> |
|ResolutionDelegate| \<Class\> |
|\<Bool\> | \<Class\> |
|\<Int> | \<Class\> | 

Ou seja, _Struct_, _Enum_, _Protocol_, _Int_, _Bool_, dentre outros, são armazenados na ***Stack*** da memória, portanto, são **mais perfomárticos** pois o tempo de acesso à Stack é **O(1)**, contudo, podem perder perfomance caso hajam propriedades que são armazenadas no _Heap_. Além disso, ***são estáticas e alocadas em tempo de compilação.***

Já as ***Classes*** são sempre armazenadas no ***Heap***, onde cada bloco, possui o tamanho das _n_ variáveis acrescidos do espaço para guardar o contador de referências e o endereço da memória. Devido a isso, são menos performáticas pois há um custo de gestão das escritas/acessos, alocação/desalocação e multithreads no _Heap_. ***São dinâmicas e alocadas em tempo de execução.***


## 3) **Gerenciamento de Memória**

Gerenciamento de memória é um processo de alocação de memória durante a compilação do programa/app ou em tempo de execução. A Memória pode ser liberada quando uma tarefa é terminada. Sabe-se quer um bom código usa o mínimo possível de memória.

Há muitos problemas durante o processo dee alocação de memória, por exemplo, o iOS liberar ou sobrescrever a memória enquanto ela ainda está em uso. Devido a isso, existem uma série de problemas que ocorrem, tais como corrupção de memória, _crashing_ do aplicativo, a aplicação corromper o dado do usuário, etc. Outro problema bastante sério é o iOS não liberar o dado ou espaço de memória que não está mais sendo usado, causando o famoso _memory leak_. 

Para tanto, o Swift fornece aos desenvolvedores ferramentas e boas práticas para evitar esses problemas.

#### 3.1) Contador Automático de Referências (_ARC_)  

O Swift usa o Contador Automático de Referências (_Automatic Reference Counting - ARC_) para observar e administrar o uso da memória do seu app. Na maioria dos casos, isso significa que o gerenciamento da memória "apenas funciona" no Swift e você não precisa se preocupar, tampouco, pensar sobre isso. O _ARC_, automaticamente, libera a memória usada por instâncias de classes quando elas não são mais necessárias.

Contudo, tem alguns casos, o _ARC_ requer mais informações sobre o relacionamento entre partes do seu código com o objetivo de gerenciar a memória para você. Abaixo, tentarei mostrar exemplo de como você habilita o _ARC_ gerenciar toda a memória do seu app.

##### _ARC_ em ação

Aqui, vou trazer o exemplo do ARC funcionando, de acordo com a documentação da Apple.

Considere o código abaixo:

```swift
class Person {
	let name: String
	init(name: String) {
		self.name = name
		print("\(name) está sendo inicializado")
	}
	deinit {
		print("\(name) está sendo deinicializado")
	}
}
```
A classe _Person_ possui um deinicializador que imprime a messagem quando uma instância da classe é desalocada (retirada da memória).

A seguir,  têm-se a definição de três variáveis do tipo _Person?_, a quais são usadas para atribuir múltiplas referências a uma nova instância da classe _Person_, lembrando que, como são opcionais, elas são inicializadas automaticamente com valor _nil_, portanto, não referem-se a uma instância de _Person_, ainda.

```swift
var reference1: Person?
var reference2: Person?
var reference3: Person?
```

Agora, você pode criar uma nova instância de _Person_ e atribuir a uma dessas 3 variáveis:

```swift
var reference1 = Person(name: "Joãozinho")
// Imprime "Joãozinho está sendo inicializado"
```

Devido a nova instância de _Person_ que foi atribuída a variável _reference1_, agora, há uma referência **forte** (***strong***) de _refereence1_ para nova instância de _Person_. Como há, pelo menos 1 referência _strong_, o _ARC_ **assegura** que essa _Person_ **permanecerá na memória** e não será desalocada.

Se você atribui a mesma instância dee _Person_ a mais duas variáveis, mais duas referências _strong_ serão estabelecidas:

```swift
var reference2 = reference1
var reference3 = reference1
```
Agora, existem **três referências forte** para uma **única instância** de _Person_

Se você quebra duas dessas referências (incluindo a referência original) pela atribuição do valor _nil_ a elas, **ainda resta uma única referência forte**, assim, **a instância de _Person_ não será desalocada**:

```swift
var reference1 = nil
var reference2 = nil
```

O _ARC_ **não desaloca** a instância de _Person_ **até que a terceira e última referência forte seja quebrada**:

```swift
var reference3 = nil
// Imprime "Joãozinho está sendo deinicializado"
```

Como você percebeu, as referências são qualificadas, o _ARC_ conta **apenas** referências **fortes (_strong_)**, então, a seguir, vamos mostrar os outros dois tipos de referências e defini-las.

#### 3.2) _Weak_ x _Unowned_

No exemplo acima, observamos que, por padrão, as referências criadas em Swift, são do tipo **_strong_**, porém, existem outros dois tipos de referências: ***weak*** e ***unowned***. A seguir, vamos defini-las e mostrar suas diferenças.

##### Referências _Weak_

De acordo com a documentação, uma referência _weak_ é uma referência que **não** mantém um controle forte da instância a que se refere e, assim, **não impede o *ARC*  de descartar a instância referenciada**. Você indica uma referência _weak_ colocando a **palavra-chave _weak_ antes da declaração da propriedade ou variável**.

Como uma referência _weak_ não mantém um controle forte da instância a que se refere, é possível que a instância seja desalocada enquanto a referência *weak* ainda esteja referenciando-a. Além, disso, **o *ARC* automaticamente atribui _nil_ à referência _weak_ quando a instância é desalocada**. E, como **referências fracas precisam alterar seus valores**, em tempo de execução, **elas sempre serão declaradas como variáveis**, em invés de constantes, de um tipo opcional.

> Observadores de propriedade não são chamados quando _ARC_ define uma referência fraca para _nil_.

Abaixo, temos um exemplo de declaração de uma referência _weak_:

```swift
class Apartment {
	let unit: String
	init(unit: String) { self.unit = unit }
	weak var tenant: Person?
	deinit { print("O apartamento \(unit) está sendo deinicializado") }
}
```

Em algum momento, você poderá se deparar com um caso de uso onde será nessário armazenar referências _weak_ em _Arrays_ ou _Dictionaries_, porém, essas estruturas apenas "aceitam" referências forte (*strong*), portanto, uma maneira de conseguir armazená-las, será criar um *wrapper* para encapsular essa referência fraca. Conforme [`WeakRef`: A Swift type-safe alternative to  `weak`properties](https://github.com/essentialdevelopercom/swift-weak-ref) podemos realizar a seguinte implementação:

```swift
final class WeakRef<T: AnyObject> {
	weak var object: T?
	
	init(_ object: T?) {
		self.object = object
	}
}
```

Podendo ser usado conforme:

```swift
class Apartment {
	let unit: String
	init(unit: String) { self.unit = unit }
	var tenant: WeakRef<Person>?
	deinit { print("O apartamento \(unit) está sendo deinicializado") }
}

var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")

john!.apartment = unit4A
unit4A!.tenant = WeakRef(john)
```

##### Referências _Unowned_

Assim como uma referência fraca (_weak_), uma referência _unowned_ **não** mantém um controle forte sobre a instância a que se referem. Porém, diferente de uma referência *weak*, uma referência *unowned* é usada quando outra instância tem o mesmo ou mais longo tempo de vida. Você define uma referência _unowned_ através do palavra-chave _unowned_ antes da declaração da propriedade ou variável.

Diferente de uma referência fraca, uma referência quando marcada como _unowned_ espera-se que **sempre** tenha um valor. Assim, marcar uma referência como _unowned_ não a faz opcional e o _ARC_ nunca atribui o valor _nil_ a uma referência _unowned_.

> Use uma referência _unowned_ **apenas quando tiver certeza** de que a referência sempre se refere a uma **instância que não foi desalocada**.
> 
> Se você tentar acessar o valor de uma referência _unowned_ depois que a instância foi desalocada, você receberá um erro de tempo de execução.

Portanto, é fortemente aconselhável utilizar referências _weak_ ao invés de _unowned_ e realizar o _safe unwrap_ para evitar _crashes_ inesperados na aplicação.

Para concluir, abaixo, uma tabela resumindo as propriedades de cada tipo de referência:

| 		    | _var_ | _let_ | opcional | não opcional |
|:---------:| :------:| :-----: | :--------: | :------------: |
|_strong_	| 👍      | 👍     | 👍         | 👍             |
| _weak_ 	| 👍      | 👎     | 👍         | 👎             |
| _unowned_ | 👍      | 👍     | 👎         | 👍             |

#### 3.3) Ciclos de Referências Fortes

Nos exemplos acima, o _ARC_ é capaz de contar o número de referências de uma nova instância de _Person_ que você cria e desalocada a instância que não está mais sendo utilizada. Contudo, é possível escrever um código, no qual, uma instância de uma classe **nunca** chega no ponto onde o contador de referências fortes é zero. Isso pode acontecer se duas classes controlam referências fortes entre elas, onde cada instância permanece "viva". Isso é conhecido como Ciclo de Referências Fortes (_strong references cycle_).

A seguir, tem-se um exemplo de  Ciclo de Referências Fortes que pode ser criado por acidente:

```swift
class Person {
	let name: String
	init(name: String) { self.name = name }
	var apartment: Apartment?
	deinit { print("\(name) está sendo deinicializado") }
}

class Apartment {
	let unit: String
	init(unit: String) { self.unit = unit }
	var tenant: Person?
	deinit { print("O apartamento \(unit) está sendo deinicializado") }
}
``` 

Toda instancia de _Person_ tem a propriedade _name_ do tipo _String_ e uma propriedade opcional _apartment_, inicialmente, _nil_.  Da mesma maneira, toda instância de _Apartment_ possui uma propriedade _unit_ do tipo _String_ e uma outra opcional, _tenant_ inicialmente _nil_ do tipo _Person_.

O próximo trecho de código, define duas variáveis opcionais, _john_ e _unit4A_, a quais serão atribuídas a uma instância de _Person_ e _Apartment_ respectivamente.

```swift
var john: Person?
var unit4A: Apartment?

john = Person(name: "Joãozinho da Silva")
unit4A = Apartment(unit: "4A")
``` 

Aqui é como as referências fortes aparecem após criar e atribuir as duas instâncias.

![](https://docs.swift.org/swift-book/_images/referenceCycle01_2x.png) Fonte: [Documentação Apple](https://docs.swift.org/swift-book/_images/referenceCycle01_2x.png)

Agora, podemos vincular essas duas referências, pois uma pessoa possui um apartamento e, portanto, um apartamento possui um inquilino.


```swift
john!.apartment = unit4A
unit4A!.tenant = john
``` 

Assim, após vincular as referências, teremos:

![](https://docs.swift.org/swift-book/_images/referenceCycle02_2x.png)
Fonte: [Documentação Apple](https://docs.swift.org/swift-book/_images/referenceCycle02_2x.png)

Infelizmente, vinculando essas duas referências, cria-se um ciclo de referências forte entre elas. Se, quebrarmos as referências controladas pelas variáveis _john_ e _unit4A_, o contador de referências **não** vai a zero e, assim, as instâncias **não são retiradas da memória** pelo _ARC_.

```swift
john = nil
unit4A = nil
``` 

Pecebe-se que nenhum deinicializador seja da classe _Person_ ou mesmo da _Apartment_, foi chamado quando atribuiu-se _nil_ às variáveis. **O ciclo de referências fortes evita que ambas instâncias sejam retiradas das memória, causando um vazamento de memória (_memory leak_) na sua aplicação**.

Aqui, é mostrado como as referências fortes aparecem após atribuir _nil_ as variáveis _john_ e _unit4A_:

![](https://docs.swift.org/swift-book/_images/referenceCycle03_2x.png)
Fonte: [Documentação Apple](https://docs.swift.org/swift-book/_images/referenceCycle03_2x.png)

As referências fortes entre a instância Person e a instância Apartment permanecem e não podem ser quebradas.

Para resolver esse problema, podemos usar a abordagem mostrada na seção anterior, marcando uma das referências como fracas, por questões de lógica, a referência que deverá ser marcada como _weak_ é a propriedade _tenant_, pois é claro que o "relacionamento" entre um apartamento e o seu inquilino não é forte, dessa forma, podemos reescrever o código realizando essa alteração e utilizando o _wrapper_ `WeakRef`:

```swift
class Apartment {
	let unit: String
	init(unit: String) { self.unit = unit }
	var tenant: WeakRef<Person>?
	deinit { print("O apartamento \(unit) está sendo deinicializado") }
}
```

Então, teremos a seguinte configuração:

![](https://docs.swift.org/swift-book/_images/weakReference01_2x.png)
Fonte: [Documentação Apple](https://docs.swift.org/swift-book/_images/weakReference01_2x.png)

Para maiores informações de como tratar casos como esses e possíveis _memories leaks_ caudados por _closures_ acessar: [https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID52](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID52)

### 4) Referências

[Memory Management in Swift: Heaps & Stacks](https://heartbeat.fritz.ai/memory-management-in-swift-heaps-stacks-baa755abe16a)

[Memory Management and Performance of Value Types](https://swiftrocks.com/memory-management-and-performance-of-value-types.html)

[https://developer.apple.com/videos/play/wwdc2016/416/](https://developer.apple.com/videos/play/wwdc2016/416/)

[Stop Using Structs!](https://medium.com/commencis/stop-using-structs-e1be9a86376f)

[Clean iOS Architecture pt.4: Clean Memory Management in Swift with WeakRef](https://medium.com/essential-developer-ios/clean-ios-architecture-pt-4-clean-memory-management-in-swift-with-weakref-1ca1501a7e35)

[Manual Memory Management](https://developer.apple.com/documentation/swift/swift_standard_library/manual_memory_management)

[Value Types and Reference Types in Swift](https://www.vadimbulavin.com/value-types-and-reference-types-in-swift/)

[Structures and Classes](https://docs.swift.org/swift-book/LanguageGuide/ClassesAndStructures.html)

[Choosing Between Structures and Classes](https://developer.apple.com/documentation/swift/choosing_between_structures_and_classes)

[Automatic Reference Counting](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)

[ARC and Memory Management in Swift](https://www.raywenderlich.com/966538-arc-and-memory-management-in-swift#toc-anchor-004)

[Memory Management](https://www.swiftbysundell.com/basics/memory-management/)

[Resource Management and Security issues in Mobile Phone Operating Systems: A Comparative Analysis](https://www.researchgate.net/publication/345691012_Resource_management_and_security_issues_in_mobile_phone_operating_systems_A_comparative_analysis)

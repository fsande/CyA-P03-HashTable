# Práctica 03. Tablas hash

### Objetivos
Los objetivos de esta práctica son: 

* Practicar conocimientos de programación Orientada a Objetos en C++
* Practicar operaciones de entrada/salida (E/S) en ficheros de texto
* Ser capaz de desarrollar, compilar y ejecutar programas escritos en C++ en el entorno de trabajo IaaS
* Profundizar en los conocimientos de Visual Studio Code (VSC)

### Rúbrica de evaluacion de esta práctica
Se señalan a continuación los aspectos más relevantes (la lista no es exhaustiva)
que se tendrán en cuenta a la hora de evaluar esta práctica:

* El comportamiento del programa debe ajustarse a lo solicitado en este documento.
* El programa ha de ser fiel al paradigma de programación orientada a objetos (OOP).
* Ha de acreditarse capacidad para introducir cambios en el programa desarrollado.
* El alumnado ha de acreditar que es capaz de editar ficheros remotos en la VM de la asignautra usando VSC
* Modularidad: el programa ha de escribirse de modo que las diferentes funcionalidades
que se precisen sean encapsuladas en funciones y/o métodos cuya extensión textual se mantenga acotada.
* El código ha de estar escrito de acuerdo al estándar de la [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html).
* El programa desarrollado deberá compilarse utilizando la herramienta `make` y un fichero `Makefile`.

Si el alumnado tiene dudas respecto a cualquiera de estos aspectos, debiera acudir al
foro de discusiones de la asignatura para plantearlas allı́. 
Se espera que, a través de ese foro, el alumnado intercambie experiencias y conocimientos, ayudándose mutuamente
a resolver dichas dudas. 
También el profesorado de la asignatura intervendrá en las discusiones que pudieran suscitarse, si fuera necesario.
    
### Introducción
Una [tabla hash](https://en.wikipedia.org/wiki/Hash_table) o tabla de dispersión es una estructura de datos que asocia claves con valores. 
La operación principal que soporta de manera eficiente es la búsqueda: permite el acceso a los valores (teléfono y dirección, por ejemplo) 
almacenados a partir de una clave generada (usando el nombre o número de cuenta, por ejemplo). 
Funciona transformando la clave mediante una [función hash](https://en.wikipedia.org/wiki/Hash_function)
en un número (hash) que identifica la posición en que la tabla hash almacena el valor correspondiente
a la clave.

Las tablas hash se suelen implementar mediante vectores unidimensionales, aunque hay otras implementaciones posibles. 
Como en el caso de los arrays, las tablas hash permiten tiempo de búsqueda constante, en promedio O(1)
independientemente del número de elementos almacenados en la tabla. 

Comience por visualizar [este vídeo](http://www.upv.es/visor/media/5199057f-ae11-a340-b7b1-50fc9da12159/c) que
introduce los conceptos más relevantes de las tablas hash.

### Ejercicio práctico
Diseñar e implementar un programa `words_in_file.cc` en C++ que lea todas las palabras de un fichero de texto y las incluya en una tabla hash.
De un segundo fichero de texto leerá una lista de palabras y escribirá en un tercer
fichero la misma lista de palabras cada una de las cuales irá seguida de un texto `Sí / No` que indicará si la
palabra en cuestión se encontraba en el fichero de entrada.

El programa se ejecutará como:

`./words_in_file infile.txt words.txt outfile.txt`

Donde `infile.txt` es el fichero que contiene el texto original, `words.txt` es el fichero que contiene la
lista de palabras (una palabra por línea) a buscar y `outfile.txt` es el fichero de salida en el que el
programa indica si cada una de las palabras de `words.txt` se encontraba o no en el texto de entrada.

Se considerará una palabra cualquier secuencia de caracteres delimitados por separadores. 
Se considerarán separadores los siguientes caracteres: espacio(' '), nueva línea ('\n'), 
tabulador ('\t'), salto de página ('\f'), retorno de carro ('\r'), coma (','), punto ('.'), 
punto y coma (';') o dos puntos (':'). 

Los separadores no pueden formar parte de las palabras, y al procesar el fichero de texto, deberán ignorarse. 
Estudiar la información (man) correspondiente a las funciones 
[`isalpha()`](https://en.cppreference.com/w/cpp/string/byte/isalpha), 
[`isalnum()`](https://en.cppreference.com/w/cpp/string/byte/isalnum) y similares de C++, por 
si resultara conveniente utilizarlas. 
Asimismo se recomienda estudiar la información de la clase [string](https://en.cppreference.com/w/cpp/string/basic_string).
Particularmente pueden resultar de interés los métodos que permiten comparar y copiar cadenas.

El tamaño de la tabla hash vendrá dado por una constante `kSizeTable`, que debe ser un número primo.

La función hash a utilizar será del tipo módulo: a cada palabra se le asociará un número, 
y el valor hash para la palabra será el resto de dividir dicho número por el tamaño de la tabla. 
Para asociar el número a la palabra, se sumarán los valores numéricos (códigos ASCII) 
correspondientes a cada uno de sus caracteres, ponderando estas sumas
([*character folding*](https://en.wikipedia.org/wiki/Hash_function#Character_folding)).

La tabla consistirá en un vector (`std::array`) de tamaño fijo (`kSizeTable`), cada uno de cuyos elementos será un puntero a una lista enlazada.
En este caso las colisiones se resolverán mediante `encadenamiento directo`: cuando a dos
palabras les corresponda la misma posición en la tabla hash, las palabras deberán aparecer encadenadas
en la lista apuntada por el puntero correspondiente a esa posición, tal como muestra 
[esta figura](https://raw.githubusercontent.com/fsande/CyA-P03-HashTable/master/tabla_hash.png).

Para la realización del programa necesitará apoyarse en sendas clases `HashTable` y `List`.
La clase `HashTable` ha de contener al menos dos atributos: el tamaño de la tabla y un puntero a listas, llamado `dataTable`. 
Este puntero se utilizará para construir, de forma dinámica un vector de listas, es
decir, un vector donde cada uno de sus elementos será un objeto lista.

Algunos de los métodos que suministra esta clase han de ser al menos el constructor, el destructor y métodos
para insertar, buscar y eliminar una palabra de la tabla.
El método `Hash` es el encargado de calcular la posición que le corresponde
en la tabla a la cadena que se le pasa como parámetro.

Obsérvese que algunos de los métodos de la clase `HashTable` (por ejemplo `find`) llevan el cualificador `const`. 
Recuerde que cuando un método está cualificado como `const` ese método no puede modificar ninguno de los atributos de la clase. 
Por otra parte, un método cualificado `const` puede ser invocado por objetos constantes y no constantes.

El único atributo de una lista es un puntero `p_start` a elementos siendo los elementos
la estructura (`struct` que consta de un objeto string para almacenar la cadena,
un campo numérico `num` en el que se puede almacenar cualquier valor numérico asociado con la cadena, por
ejemplo el número de apariciones de la cadena en el texto, y un puntero al siguiente nodo de la lista.

Para el desarrollo de la práctica ha de implementar al menos los métodos descritos anteriormente.
El programa `words_in_file` es un programa cliente (hace uso) de las clases `HashTable` y `List`.
Este programa:

* Leerá desde la línea de comandos tres nombres de fichero
* Si en la línea de comandos no se suministra esta información, se imprimirá un mensaje de error indicando
  el modo correcto de ejecución del programa y se finalizará la ejecución
* Se abrirá el primer fichero, y se leerán todas las palabras que contenga, insertándolas en la tabla hash
* A continuación se leerán las palabras del segundo fichero y se buscarán en la tabla hash, imprimiendo en el
* tercer fichero (de salida) la indicación de si la palabra en cuestión se encuentra o no en el primero

Se muestra a continuación y exclusivamente a título informativo algunas posibles definiciones para las clases
que han de desarrollarse así como algunos métodos que cabría encontrar en esas clases.

```cpp
struct elem{string str; unsigned num; elem *next;};
 
class List {
 public:
   List();
   ~List();
   void RemoveNode(const string str);
   void ListInsert(const string str, unsigned num);
   elem *FindPosition(const string str) const;
   void Remove(elem *p);
 private:
   elem *p_start;
 };
 
 class HashTable {
 public:
   HashTable(unsigned len = kSizeTable);
   ~HashTable();
   void Insert(const string str, unsigned num);
   elem *Find(const string str) const;
   void Remove(const string str);
   unsigned Hash(const string str) const;
 private:
   unsigned size_table;
   List *data_table;
 };
```

Se enumeran a continuación los pasos que habitualmente se siguen cuando se utiliza una metodología orientada
a objetos en el diseño de un programa:
* En el ámbito de aplicación de su programa, identifique las entidades (objetos) y sus atributos (datos).
* Determine las acciones que pueden realizarse sobre un objeto.
* Determine las acciones que un objeto puede realizar sobre otros objetos.
* Determine las partes de cada objeto que serán visibles a otros objetos, qué partes serán públicas y cuáles privadas.
* Defina la interfaz pública de cada objeto.

Como mejora opcional para la implementación de esta práctica, se propone utilizar 
programación genérica (a través de plantillas, *Templates*) para tratar de diseñar clases que soporten tablas hash y listas en 
las que se pueda almacenar diferentes tipos de estructuras de datos.

### Referencias
* [Hash table](https://en.wikipedia.org/wiki/Hash_table)
* [Hash function](https://en.wikipedia.org/wiki/Hash_function)
* [Vídeo Tablas Hash](http://www.upv.es/visor/media/5199057f-ae11-a340-b7b1-50fc9da12159/c) Universidad
 Politécnica de Valencia
* [String class](http://www.cplusplus.com/reference/string/string/)
* [Standard Template Library](http://www.cplusplus.com/reference/stl/)
* [C++ Tutor](http://pythontutor.com/cpp.html#mode=display) Visualización online de la ejecución de código C++
* [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html) Guía de estilo de código 

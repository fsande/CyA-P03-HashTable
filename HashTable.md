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
El programa se ejecutará como:

`./words_in_file infile.txt`

y en caso de ejecutarse sin pasar argumentos en la línea de comandos, debe imprimir en pantalla un mensaje
indicando la forma correcta de uso.

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
en la lista apuntada por el puntero correspondiente a esa posición, tal como muestra la figura \ref{fig:tabla}.
%%%%%%%%%%%%%%%%%%%%% Fig. %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
\centerline{\includegraphics[width=10cm]{tabla_hash}}
\caption{Estructura de la tabla hash}
\label{fig:tabla}
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

La figura \ref{code:hash.h} muestra el contenido del fichero `hash.h` en el
que se definen la clases `HashTable` y `list`. 

La clase `HashTable` contiene 2 atributos privados: el tamaño de la tabla 
y un puntero a listas, llamado `tabla`. 
Este puntero se utilizará para construir, de forma dinámica un vector de listas, es
decir, un vector donde cada uno de sus elementos será un objeto lista.

Los métodos que suministra esta clase son el constructor, el destructor y métodos
para insertar, buscar y eliminar una palabra de la tabla.

El método `hash` es el encargado de calcular la posición que le corresponde
en la tabla a la cadena que se le pasa como parámetro.

Obsérvese que algunos de los métodos de la clase `HashTable` (por ejemplo `find`,
definido en la línea 30) llevan el cualificador `const`. 
Recuerde que cuando un método está cualificado como `const` el método no puede modificar ninguno de los atributos
de la clase. 
Por otra parte, un método cualificado `const` puede ser invocado por objetos
constantes y no constantes.

El fichero `hash.h` también contiene las definiciones correspondientes a la clase
`list`, que se utiliza para gestionar las listas de elementos a las que apuntan las 
casillas de la tabla hash.

El único atributo de una lista es un puntero `pStart` a elementos siendo los elementos
la estructura que se define en la línea 4, que consta de un puntero a caracteres (para almacenar la cadena),
un campo numérico `num` en el que se puede almacenar cualquier valor numérico asociado con la cadena, por
ejemplo el número de línea en que la cadena aparece en el texto, y un puntero
al siguiente nodo de la lista.

Para la implementación de esta práctica lo/as alumno/as deberán codificar el contenido
del fichero `hash.C`. Este fichero deberá contener la implementación de todos
los métodos definidos en `hash.h`.

Para probar el programa debe diseñarse también un programa cliente que:
\begin{enumerate}
\item Leerá desde la línea de comandos un nombre de fichero
\item Si en la línea de comandos no se suministra un nombre de fichero, se imprimirá un mensaje de error
y se finalizará la ejecución
\item Se abrirá el fichero, y se leerán todas las palabras que contenga, insertándolas en la tabla hash
\item Después el programa entrará en un bucle en el que pedirá al usuario una palabra y le indicará
si la palabra estaba o no en el fichero, buscándola en la tabla hash
\item El programa finalizará cuando el usuario introduzca la palabra `FIN` escrita en mayúsculas
\end{enumerate}

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
\begin{lstlisting}[caption={El fichero hash.h},label=code:hash.h]     
1  #include <iostream.h>
2  #include <stdlib.h>
3  
4  struct elem{char *name; unsigned num; elem *next;};
5  
6  class list {
7  public:
8     list();
9     ~list();
10    void RemoveNode(const char *s);
11    void ListInsert(const char *s, unsigned num);
12    elem *FindPosition(const char *s)const;
13    void remove(elem *p);
14    list(const list& s) {
15      cout << "La copia no está permitida." << endl; exit(1);
16    }
17    list &operator=(const list &s) {
18      cout << "La asignación no está permitida." << endl; exit(1);
19      return *this;
20    }
21 private:
22    elem *pStart;
23 };
24 
25 class HashTable {
26 public:
27    HashTable(unsigned len=1021);
28    ~HashTable();
29    void insert(const char *s, unsigned num);
30    elem *find(const char *s) const;
31    void remove(const char *s);
32    unsigned hash(const char *s) const;
33 private:
34    unsigned size_table;
35    list *tabla;
36 };
\end{lstlisting}
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

Enumeramos a continuación los pasos que habitualmente se siguen cuando se utiliza una metodología orientada
a objetos en el diseño de un programa:
\begin{enumerate}
\item En el ámbito de aplicación de su programa, identifique las entidades (objetos) y sus atributos (datos).
\item Determine las acciones que pueden realizarse sobre un objeto.
\item Determine las acciones que un objeto puede realizar sobre otros objetos.
\item Determine las partes de cada objeto que serán visibles a otros objetos, qué partes serán públicas y cuáles
privadas.
\item Defina la interface pública de cada objeto.
\end{enumerate}

Como mejora opcional para la implementación de esta práctica, l@s alumn@s podrían estudiar los aspectos comunes
que encuentren entre ésta implementación y la que realizaron en la práctica anterior, y utilizando
programación genérica tratar de diseñar una clase que soporte tablas hash en las que se pueda almacenar
diferentes tipos de estructuras de datos.

Otra posibilidad es implementar, también utilizando programación genérica una clase lista en la que se pueda almacenar
diferentes tipos de elementos.
\end{document}




### Referencias
* [Hash table](https://en.wikipedia.org/wiki/Hash_table)
* [Hash function](https://en.wikipedia.org/wiki/Hash_function)
* [Vídeo Tablas Hash](http://www.upv.es/visor/media/5199057f-ae11-a340-b7b1-50fc9da12159/c) Universidad
 Politécnica de Valencia

* [String class](http://www.cplusplus.com/reference/string/string/)

* [Standard Template Library](http://www.cplusplus.com/reference/stl/)
* [Fundamental types](https://en.cppreference.com/w/cpp/language/types)
* [Bitwise Operators in C and C++](https://www.cprogramming.com/tutorial/bitwise_operators.html)
* [C++ Tutor](http://pythontutor.com/cpp.html#mode=display) Visualización online de la ejecución de código C++
* [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html) Guía de estilo de código 

# Sistema de Gestión de Misiones de Rescate (SGMR)

**Curso:** Fundamentos de Programación Orientada a Objetos – Grupo 80
**Evaluación:** Parcial 2


* John Santiago Jurado 2537685

## 1\. Descripción

Motor lógico en C++ para la gestión de recursos humanos y vehiculares de la Cruz Roja
durante emergencias en el Valle del Cauca. Permite registrar personal y vehículos, crear
misiones de emergencia, asignarles recursos de forma polimórfica y ejecutar la acción
operativa de cada recurso mediante una interfaz de consola.

## 2\. Arquitectura y jerarquía de clases

```
Recurso (abstracta)
├── Vehiculo (abstracta)
│   ├── Ambulancia
│   └── Helicoptero
└── Personal (abstracta)
    ├── Medico
    └── Rescatista

Mision      -> agrega (0..\*) Recurso\*   (no es dueña de la memoria)
Controlador -> administra (0..\*) Recurso\* y (0..\*) Mision\*  (dueño de la memoria)
main.cpp    -> crea 1 Controlador
```

El diagrama de clases completo está en `diagrama/diagrama\_clases\_SGMR.png`
(fuente editable en `diagrama/uml.dot`, generado con Graphviz).

### Decisiones de diseño clave

* **Recurso** es una clase abstracta (método `ejecutarAccion()` puro) que define el
contrato común usado para el polimorfismo pedido en HU02 y HU03.
* **Vehiculo** y **Personal** son clases intermedias abstractas que agrupan atributos
comunes (placa/capacidad vs. especialidad) sin implementar `ejecutarAccion()`.
* **Mision** almacena los recursos asignados en un arreglo dinámico de punteros
(`Recurso\*\* recursosAsignados`) que redimensiona manualmente al duplicarse
(sin `std::vector`, según HT01). Mision **no** libera los objetos `Recurso`
apuntados en su destructor —solo libera el arreglo de punteros— porque no es
la dueña de esa memoria; esto evita una doble liberación (double free).
* **Controlador** es la única clase dueña de la memoria de `Recurso` y `Mision`
(los crea con `new` y los libera con `delete`/`delete\[]` en su destructor,
cumpliendo HT02).
* Todo paso de objetos complejos entre métodos se hace por puntero o referencia
(HT03), nunca por copia en la pila.
* `main.cpp` solo instancia `Controlador` e invoca `iniciar()` (HU04).
* Al construirse, `Controlador` llama automáticamente a `cargarDatosPrueba()`
(HU05), dejando precargados 2 misiones, 2 ambulancias, 1 helicóptero,
2 médicos y 2 rescatistas.

## 3\. Estructura del repositorio

```
SGMR/
├── include/          # Archivos .h (declaraciones)
├── src/               # Archivos .cpp (implementación) + main.cpp
├── diagrama/          # Diagrama de clases UML (.png y .dot)
├── README.md
└── .gitignore
```

## 4\. Compilación y ejecución

Requiere un compilador con soporte C++11 (g++ recomendado).

```bash
g++ -Wall -std=c++11 src/\*.cpp -o sgmr
./sgmr
```

## 5\. Menú de la aplicación

```
1. Ver recursos y misiones
2. Registrar recurso
3. Crear misión
4. Asignar recurso a misión
5. Ejecutar misión
6. Salir
```

## 6\. Verificación de memoria

El proyecto fue probado con `valgrind --leak-check=full`, confirmando:
`All heap blocks were freed -- no leaks are possible` y `0 errors`.

## 7\. Trazabilidad con los requerimientos

|Requerimiento|Dónde se resuelve|
|-|-|
|HU01 (Registro)|`Controlador::registrarRecurso()`|
|HU02 (Asignación polimórfica)|`Mision::asignarRecurso(Recurso\*)`|
|HU03 (Ejecución polimórfica)|`Mision::ejecutarMision()` + `ejecutarAccion()` virtual en cada subclase|
|HU04 (Menú y Controlador)|`main.cpp` + `Controlador::iniciar()`|
|HU05 (Datos quemados)|`Controlador::cargarDatosPrueba()`|
|HT01 (Memoria dinámica manual)|Arreglos `Recurso\*\*` / `Mision\*\*` con redimensionamiento manual, sin `std::vector`|
|HT02 (Destrucción segura)|Destructores de `Controlador`, `Mision` y jerarquía `Recurso`|
|HT03 (Paso eficiente de mensajes)|Todos los métodos reciben objetos por `\&` o `\*`|




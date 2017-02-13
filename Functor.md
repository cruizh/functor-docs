# Funtores
Los funtores son una herramienta fundamental para la programación de orden superior.
Se utilizan para introducir contexto en la programación, por ejemplo,
la posibilidad de tratar con errores o de tener estructuras de datos complejas.
Se construyen sobre una generalización de la función `map :: (a -> b) -> [a] -> [b]` ya conocida,
y sirven de base del razonamiento funcional.

El funtor es una clase (una categoría que los tipos satisfacen por tener definidas unas funciones determinadas)
determinada por la función `fmap`.
Así, a efectos prácticos, podemos definir funtor como sigue:

``` haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

Nótese que `Functor` está definido sobre `f`, que es un constructor de tipos.
Algunos ejemplos de constructores de tipo son `Maybe`, `Either`, árboles, listas, etc.
`Functor` está definido para cada constructor de tipos de forma general, y no para cada tipo concreto,
es decir, basta una sola definición de para cualquier tipo de la forma `Maybe a`, otra para cualquier tipo de la forma
`Either a b`, y otra para cualquier tipo de la forma `[a]`.

Como hemos mencionado anteriormente, `fmap` es una generalización de `map`, por lo que la instancia de `Functor` para las listas es sencilla:

``` haskell
instance Functor [] where
    fmap = map
```

Consideremos para facilitar la comprensión del concepto que los constructores de tipos actúan como cajas.
Por ejemplo, las listas serían cajas con varios elementos,
y los valores del tipo `Maybe a` serían cajas que podrían contener un valor del tipo `a` o estar vacías.
Así, podemos describir `fmap` como una función que toma una función que actúa sobre valores concretos y la aplica sobre valores que están dentro de una caja, volviendo a meter los valores en las cajas al terminar.
Veamos cuál sería la instancia de `Functor` para los valores `Maybe`, es decir, cómo aplicaríamos funciones ordinarias cuando está presente la "caja" `Maybe`.
Si se trata de un valor `Just x`, aplicaríamos la función directamente, pero si la caja está vacía (el valor especial `Nothing`), no tenemos nada a lo que aplicar la función, por lo que no tenemos nada que devolver. Formalmente:

``` haskell
instance Functor Maybe where
    fmap f (Just x) = Just (f x)
    fmap _ Nothing = Nothing
```

Supongamos que tenemos un tipo árbol binario como el siguiente:

``` haskell
data Arbol a = Hoja a
             | Nodo a (Arbol a) (Arbol a)
```

¿Cómo se definiría en este caso `fmap`? Muy sencillo:

``` haskell
instance Functor Arbol where
    fmap f (Hoja x) = Hoja (f x)
    fmap f (Nodo x i d) = Nodo (f x) (fmap f i) (fmap f d)
```

Existe una función infija sinónima de `fmap`:

``` haskell
(<$>) = fmap
```

Esta función es asociativa por la izquierda y nos servirá más adelante.

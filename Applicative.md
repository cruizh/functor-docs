# Funtores aplicativos

Los funtores solo permiten aplicar funciones de un argumento a un valor con contexto.
Si queremos dotar de contexto a funciones con un número arbitrario de argumentos, necesitamos
de una herramienta más potente. Es el caso de los funtores aplicativos.
Los funtores aplicativos son un tipo de funtores que además tienen aplicación,
es decir, permiten dotar de contexto a la computación propiamente dicha.
Los funtores aplicativos permiten dotar de un contexto básico a expresiones puras y
secuenciar computaciones. Veamos la definición y un ejemplo:

``` haskell
class Functor f => Applicative f where
    pure :: a -> f a
    (<*>) :: f (a -> b) -> f a -> f b
```

La función `pure` dota de un contexto simple a un valor o una función, mientras
que la función `(<*>)` toma el papel análogo a `fmap`. Veamos una instancia:

``` haskell
instance Applicative Maybe where
    pure = Just
    
    Just f  <*> m = fmap f m
    Nothing <*> _ = Nothing
```

Vemos que `pure` es obvio, así que fijémonos en `(<*>)`. Existen dos posibilidades:
1. Si hay una función f, entonces se aplica al valor contenido en `m :: Maybe a`, si es que en `m` hay algo.
2. Si no hay una función que aplicar, no hay nada que hacer.

Las listas también forman funtores aplicativos. Veamos los tipos de las funciones:

``` haskell
pure :: a -> [a]
(<*>) :: [a -> b] -> [a] -> [b]
```

¿Es posible intuir las definiciones de las funciones? `pure` será una función
que transforme un elemento en una lista con ese elemento, mientras que `(<*>)`
será una función que tome una lista de funciones y la aplique a una lista de elementos.
Veamos la definición:

``` haskell
instance Applicative [] where
    pure x = [x]
    fs <*> xs = [f x | f <- fs, x <- xs]
```

¡Eureka! Las listas por comprensión son también funtores aplicativos.
Gracias a la notación infija, podemos extenderlo a funciones que toman más de un argumento:

``` haskell
fs <*> xs <*> ys == (fs <*> xs) <*> ys == [f x | f <- fs, x <- xs] <*> ys [f x y | f <- fs, x <- xs, y <- ys]
```

Como los funtores aplicativos son funtores, podemos combinar las operaciones. Consideremos lo siguiente:

``` haskell
g :: a -> b -> c
x :: f a
y :: f b
```
Veamos qué hace `g <$> x <*> y` (que es equivalente a `(g <$> x) <*> y`).
Estudiemos los tipos:

``` haskell
g :: a -> (b -> c)
g <$> x == fmap g s :: f (b -> c)
(<*>) :: f (b -> c) -> f b -> f c
(g <$> x) <*> y :: f c
```

Luego los funtores aplicativos permiten aplicar funciones de varios argumentos a valores con contexto,
y `(<*>)` permite aplicar funciones con contexto a valores con contexto.
El módulo `Control.Applicative` define una función `liftA2 :: Applicative f => f (a -> b -> c) -> f a -> f b -> f c`
que hace lo mismo que hemos hecho "a mano" con `g <$> x <*> y` para el caso de funciones de 2 argumentos,
y análogamente una función `liftA3` para el caso de 3 argumentos.
También define una función `liftA f a = pure f <*> a` para funciones de un solo argumento, equivalente a `fmap`.

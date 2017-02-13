# Monoides #

Al igual que definimos las clases `Eq` para tipos cuyos valores pueden ser comparados por igualdad,
`Ord` para tipos cuyos valores pueden ser ordenados
y `Functor` para valores que pueden ser tratados como "cajas" o contextos computacionales,
podemos definir clases para las estructuras algebraicas. Una de las estructuras más útiles son los monoides.

Un grupo es un par _(Tipo, Operación)_, donde la operación es interna, asociativa, tiene elemento neutro y tiene elemento opuesto.
Si no exigimos la existencia de elemento opuesto, la estructura resultante
(la operación es interna, asociativa y tiene elemento neutro) se llama monoide.

Podemos expresar esta estructura en el sistema de tipos de Haskell para definir funciones que actúen sobre monoides:

``` haskell
class Monoid m where
    mempty :: m -- Elemento neutro
    mappend :: m -> m -> m -- la operación interna
    
    mconcat :: [m] -> m
    mconcat = foldr mappend mempty
```

También está definido `(<>) = mappend`, un sinónimo infijo.

La clase `Monoid` está definida en `Data.Monoid`.
Como la definición de `mconcat` sólo depende de `mempty` y `mappend`, basta definir el elemento neutro y la operación.
Veamos las propiedades de los monoides expresadas en Haskell:

``` haskell
x <> mempty = x
mempty <> x = x
(x <> y) <> z == x <> (y <> z)
```

Veamos algunos ejemplos de monoides:

``` haskell
instance Monoid [a] where
    mempty = []
    mappend = (++)
```

En el caso de los números, existen varias posibilidades.
Por simplificar, consideremos los enteros.
Los enteros forman monoides la suma y el 0, o con la multiplicación y el 1
(nótese que los naturales forman monoides con las mismas operaciones).
Cuando existen varios monoides que se pueden aplicar al mismo tipo,
podemos definir un `newtype`.
Un `newtype` es un caso especial de `data` en el que solo existe _un_ constructor
con _un_ campo, i.e. no lo podemos usar para definir `Maybe a` o `Either a b`.
Esta restricción permite que los tipos sean isomorfos y que sean tratados como el mismo tipo.
Esto nos permite tratar los dos monoides anteriores de forma separada,
manteniendo el resto de operaciones típicas de los enteros:

``` haskell
newtype Sum a = Sum { getSum :: a }
    deriving (Eq, Ord, Read, Show, Bounded)

newtype Product a = Product { getProduct :: a }
    deriving (Eq, Ord, Read, Show, Bounded)


instance Num a => Monoid (Sum a) where
    mempty = Sum 0
    Sum x `mappend` Sum y = Sum (x+y)

instance Num a => Monoid (Product a) where
    mempty = Product 1
    Product x `mappend` Product y = Product (x*y)
```

Así, podemos tratar sumas y productos de forma general. Por ejemplo:

``` haskell
Sum 2 <> Sum 3 == Sum 5
sum = getSum . mconcat . map Sum
product = getProduct . mconcat . map Product
```

Otro caso es el de los valores lógicos. Los booleanos forman monoides con los operadores O e Y:

``` haskell
newtype Any = Any { getAny :: Bool }
    deriving (Eq, Read, Show)

newtype All = All { getAll :: Bool }
    deriving (Eq, Read, Show)

instance Monoid Any where
    mempty = Any False
    Any False `mappend` Any False = Any False
    _ `mappend` _ = Any True

instance Monoid All where
    mempty = All True
    All True `mappend` All True = All True
    _ `mappend` _ = All False
```

Una clase donde los monoides son muy útiles es `Foldable`, definida en `Data.Foldable`.
Los tipos `Foldable` se basan en la conocida función `foldr`. Si el tipo es un monoide,
se puede definir simplemente una función `foldMap` que equivale a `foldr` como sigue:

``` haskell
class Foldable t where
    foldMap :: Monoid m => (a -> m) -> t a -> m
```

`foldMap` funciona como una generalización de `mconcat . map`.
Supongamos que tenemos un tipo árbol como sigue:

``` haskell
data Arbol a = Hoja
             | Nodo a (Arbol a) (Arbol a)
```

Podemos definir instancias de `Functor` y `Foldable` como sigue:

``` haskell
instance Functor Arbol where
    fmap _ Hoja = Hoja
    fmap f (Nodo x l r) = Nodo (f x) (fmap f l) (fmap f r)

instance Foldable Arbol where
    foldMap _ Hoja = mempty
    foldMap f (Nodo x l r) = f x <> foldMap f l <> foldMap f r
```

Así, las siguientes operaciones son equivalentes:

``` haskell
xs = [1..5]
ar = Nodo 3 (Nodo 2 (Nodo 1 Hoja Hoja) Hoja) (Nodo 4 Hoja (Nodo 5 Hoja Hoja))

bs = map (<3) xs
ba = fmap (<3) ar

-- Estas definiciones forman parte de Prelude
any = getAny . foldMap Any
all = getAll . foldMap All

all bs == all ba == getAll . foldMap (All . (<3)) ar == all bs
```

Así, podemos combinar estructuras de datos distintas a las listas.
Además, `Foldable` provee todas las funciones que se usan comumente con listas:

``` haskell
null Hoja == True
null (Nodo x Hoja Hoja) == False

0 `elem` ar == False
2 `elem` ar == True

length ar == 5
maximum ar == 5
minimum ar == 1
sum ar == 15
product ar == 120
toList ar == [3,2,1,4,5]
```


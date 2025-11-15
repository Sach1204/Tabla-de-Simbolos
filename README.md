

# Tabla de Símbolos y Código de Tres Direccion

## CÓDIGO PYTHON

###  `gramatica.py` - Definición de la Gramática

Este archivo contiene la especificación formal de la gramática que el compilador acepta:

```python
GRAMATICA = """
Programa   -> ListaSentencias EOF

ListaSentencias -> Sentencia (NEWLINE Sentencia)* NEWLINE*

Sentencia  -> ID '=' Expresion
            | 'print' '(' Expresion ')'

Expresion  -> Termino (('+' | '-') Termino)*

Termino    -> Factor (('*' | '/') Factor)*

Factor     -> NUM
            | ID
            | '(' Expresion ')'
"""
```

### `entrada.py` - Código Fuente de Entrada

Este es el programa que será compilado:

```python
x = 4
y = x * 2
z = y + x - 3
w = (z * 2) + (y / x)
print(x)
print(w)
```

**Análisis línea por línea**:

1. `x = 4` → Asigna el valor 4 a la variable x
2. `y = x * 2` → y toma el valor de x multiplicado por 2
3. `z = y + x - 3` → z es la suma de y y x, menos 3
4. `w = (z * 2) + (y / x)` → w combina multiplicación y división con precedencia
5. `print(x)` → Imprime el valor de x
6. `print(w)` → Imprime el valor de w

---

### `main.py` - Compilador Completo

#### **Sección 1: Definición de Tokens**

```python
TT_ID = "ID"
TT_NUM = "NUM"
TT_EQ = "="
TT_PLUS = "+"
TT_MINUS = "-"
TT_MUL = "*"
TT_DIV = "/"
TT_LPAREN = "("
TT_RPAREN = ")"
TT_NEWLINE = "NEWLINE"
TT_EOF = "EOF"
TT_PRINT = "PRINT"

KEYWORDS = {
    "print": TT_PRINT,
}
```

Define constantes para cada tipo de token y un diccionario de palabras reservadas.

---

#### **Clase Token**

```python
class Token:
    def __init__(self, type_, value=None, line=1, col=1):
        self.type = type_
        self.value = value
        self.line = line
        self.col = col
```

Representa un token con:
- `type`: Tipo de token (ID, NUM, PLUS, etc.)
- `value`: Valor del token (nombre de variable, número)
- `line`, `col`: Posición en el código fuente (para mensajes de error)

---

#### **Lexer (Análisis Léxico)**

```python
class Lexer:
    def __init__(self, text):
        self.text = text
        self.pos = 0
        self.line = 1
        self.col = 1
```

**Funciones principales**:

1. **`current_char()`**: Retorna el carácter actual
2. **`advance()`**: Avanza al siguiente carácter y actualiza línea/columna
3. **`skip_whitespace()`**: Ignora espacios y tabulaciones
4. **`number()`**: Reconoce números (enteros y decimales)
5. **`identifier()`**: Reconoce identificadores y palabras reservadas
6. **`get_tokens()`**: Tokeniza todo el texto


#### **Sección 4: Nodos del AST**

```python
class Node:
    pass

class Program(Node):
    def __init__(self, statements):
        self.statements = statements

class Assign(Node):
    def __init__(self, name, expr):
        self.name = name
        self.expr = expr

class Print(Node):
    def __init__(self, expr):
        self.expr = expr

class BinOp(Node):
    def __init__(self, op, left, right):
        self.op = op
        self.left = left
        self.right = right

class Num(Node):
    def __init__(self, value):
        self.value = value

class Var(Node):
    def __init__(self, name):
        self.name = name
```

Cada clase representa un tipo de nodo en el árbol:
- **Program**: Raíz del árbol (contiene todas las sentencias)
- **Assign**: Asignación (`x = expr`)
- **Print**: Impresión (`print(expr)`)
- **BinOp**: Operación binaria (+, -, *, /)
- **Num**: Literal numérico
- **Var**: Referencia a variable

---

#### **Parser (Análisis Sintáctico)**

```python
class Parser:
    def __init__(self, tokens):
        self.tokens = tokens
        self.pos = 0
```

**Métodos principales**:

1. **`parse()`**: Punto de entrada, retorna el AST completo
2. **`parse_stmt_list()`**: Procesa lista de sentencias
3. **`parse_stmt()`**: Procesa una sentencia (asignación o print)
4. **`parse_expr()`**: Procesa expresiones (suma/resta)
5. **`parse_term()`**: Procesa términos (multiplicación/división)
6. **`parse_factor()`**: Procesa factores (números, variables, paréntesis)

**Precedencia de operadores**:
- Mayor precedencia: `*`, `/` (se evalúan primero)
- Menor precedencia: `+`, `-` (se evalúan después)

---

#### **Tabla de Símbolos**

```python
def construir_tabla_simbolos(ast):
    tabla = {}
    
    def visitar(nodo):
        if isinstance(nodo, Assign):
            info = tabla.get(nodo.name)
            if info is None:
                info = {"tipo": "num", "ocurrencias": 0}
                tabla[nodo.name] = info
            info["ocurrencias"] += 1
            visitar(nodo.expr)
        
        elif isinstance(nodo, Var):
            info = tabla.get(nodo.name)
            if info is None:
                info = {"tipo": "num", "ocurrencias": 0}
                tabla[nodo.name] = info
            info["ocurrencias"] += 1
        # ... otros casos
    
    visitar(ast)
    return tabla
```

**Funcionamiento**:
- Recorre el AST recursivamente
- Registra cada variable encontrada
- Cuenta cuántas veces aparece cada variable
- Almacena el tipo (en este caso, siempre "num")


#### **Sección 7: Generador de Código de Tres Direcciones (TAC)**

```python
class TacGenerator:
    def __init__(self):
        self.temp_count = 0
        self.code = []
    
    def nuevo_temp(self):
        self.temp_count += 1
        return "t%d" % self.temp_count
```

**Métodos principales**:

1. **`generar(ast)`**: Procesa el AST completo
2. **`gen_stmt(nodo)`**: Genera código para sentencias
3. **`gen_expr(nodo)`**: Genera código para expresiones

**Funcionamiento del generador**:

```python
def gen_expr(self, nodo):
    if isinstance(nodo, Num):
        tmp = self.nuevo_temp()
        self.code.append("%s = %s" % (tmp, nodo.value))
        return tmp
    
    if isinstance(nodo, BinOp):
        left_place = self.gen_expr(nodo.left)
        right_place = self.gen_expr(nodo.right)
        tmp = self.nuevo_temp()
        self.code.append("%s = %s %s %s" % (tmp, left_place, op, right_place))
        return tmp
```

Cada operación se descompone en instrucciones simples con máximo 3 direcciones:
- Una dirección para el resultado
- Dos direcciones para los operandos

---

#### **Sección 8: Función Main**

```python
def main():
    # 1) Leer el código fuente
    with open("entrada.py", "r", encoding="utf-8") as f:
        fuente = f.read()
    
    # 2) Analizar (EDTS completa)
    ast, tabla, tac_code = analizar_codigo_fuente(fuente)
    
    # 3) Generar ast.txt
    with open("ast.txt", "w", encoding="utf-8") as f_ast:
        f_ast.write("== Gramática ==\n")
        f_ast.write(GRAMATICA)
        f_ast.write("\n\n== AST ==\n")
        f_ast.write(ast_a_string(ast))
    
    # 4) Generar tabla_simbolos.txt
    with open("tabla_simbolos.txt", "w", encoding="utf-8") as f_ts:
        f_ts.write(tabla_simbolos_a_string(tabla))
    
    # 5) Generar tac.txt
    with open("tac.txt", "w", encoding="utf-8") as f_tac:
        f_tac.write(tac_a_string(tac_code))
```

**Flujo de ejecución**:
1. Lee `entrada.py`
2. Ejecuta el pipeline completo (Lexer → Parser → Tabla → TAC)
3. Guarda los resultados en 3 archivos de texto

---

## ARCHIVOS DE SALIDA (TXT)

###  `ast.txt` - Árbol de Sintaxis Abstracta

```
== Gramática ==
[... definición de la gramática ...]

== AST ==
Program
  Assign(name=x)
    Num(4.0)
  Assign(name=y)
    BinOp(op=*)
      Var(x)
      Num(2.0)
  Assign(name=z)
    BinOp(op=-)
      BinOp(op=+)
        Var(y)
        Var(x)
      Num(3.0)
  Assign(name=w)
    BinOp(op=+)
      BinOp(op=*)
        Var(z)
        Num(2.0)
      BinOp(op=/)
        Var(y)
        Var(x)
  Print
    Var(x)
  Print
    Var(w)
```

**Análisis detallado del AST**:

#### **Sentencia 1**: `x = 4`
```
Assign(name=x)
  Num(4.0)
```
- Nodo raíz: Asignación a la variable `x`
- Hijo: Número literal `4.0`

#### **Sentencia 2**: `y = x * 2`
```
Assign(name=y)
  BinOp(op=*)
    Var(x)
    Num(2.0)
```
- Asignación a `y`
- Expresión: Multiplicación
  - Operando izquierdo: Variable `x`
  - Operando derecho: Número `2.0`

#### **Sentencia 3**: `z = y + x - 3`
```
Assign(name=z)
  BinOp(op=-)
    BinOp(op=+)
      Var(y)
      Var(x)
    Num(3.0)
```
- Asignación a `z`
- Estructura jerárquica que respeta precedencia:
  1. Primero se evalúa `y + x` (nivel interno)
  2. Luego se resta `3` (nivel externo)

#### **Sentencia 4**: `w = (z * 2) + (y / x)`
```
Assign(name=w)
  BinOp(op=+)
    BinOp(op=*)
      Var(z)
      Num(2.0)
    BinOp(op=/)
      Var(y)
      Var(x)
```
- Asignación a `w`
- Suma de dos subexpresiones:
  - Subexpresión izquierda: `z * 2`
  - Subexpresión derecha: `y / x`

#### **Sentencias 5 y 6**: `print(x)` y `print(w)`
```
Print
  Var(x)
Print
  Var(w)
```
- Dos nodos de impresión
- Cada uno imprime una variable

---

### `tabla_simbolos.txt` - Tabla de Símbolos

```
== Tabla de símbolos ==
w          tipo=num ocurrencias=2
x          tipo=num ocurrencias=5
y          tipo=num ocurrencias=3
z          tipo=num ocurrencias=2
```

**Análisis de cada variable**:

| Variable | Tipo | Ocurrencias | Desglose de apariciones |
|----------|------|-------------|-------------------------|
| **w** | num | 2 | 1. Definición: `w = ...` <br> 2. Uso: `print(w)` |
| **x** | num | 5 | 1. Definición: `x = 4` <br> 2. Uso en: `y = x * 2` <br> 3. Uso en: `z = y + x - 3` <br> 4. Uso en: `w = ... + (y / x)` <br> 5. Uso en: `print(x)` |
| **y** | num | 3 | 1. Definición: `y = x * 2` <br> 2. Uso en: `z = y + x - 3` <br> 3. Uso en: `w = ... + (y / x)` |
| **z** | num | 2 | 1. Definición: `z = y + x - 3` <br> 2. Uso en: `w = (z * 2) + ...` |

**Observaciones importantes**:
- Todas las variables son de tipo `num` (numérico)
- `x` es la variable más utilizada (5 apariciones)
- Las ocurrencias cuentan TANTO definiciones COMO usos
- Las variables están ordenadas alfabéticamente

---

### `tac.txt` - Código de Tres Direcciones

```
== Código en tres direcciones ==
t1 = 4.0
x = t1
t2 = 2.0
t3 = x * t2
y = t3
t4 = y + x
t5 = 3.0
t6 = t4 - t5
z = t6
t7 = 2.0
t8 = z * t7
t9 = y / x
t10 = t8 + t9
w = t10
print x
print w
```

**Análisis línea por línea con rastreo de valores**:

#### **Bloque 1**: `x = 4`
```
t1 = 4.0      # t1 ← 4.0
x = t1        # x ← 4.0
```
**Resultado**: `x = 4.0`

---

#### **Bloque 2**: `y = x * 2`
```
t2 = 2.0      # t2 ← 2.0
t3 = x * t2   # t3 ← 4.0 * 2.0 = 8.0
y = t3        # y ← 8.0
```
**Resultado**: `y = 8.0`

---

#### **Bloque 3**: `z = y + x - 3`
```
t4 = y + x    # t4 ← 8.0 + 4.0 = 12.0
t5 = 3.0      # t5 ← 3.0
t6 = t4 - t5  # t6 ← 12.0 - 3.0 = 9.0
z = t6        # z ← 9.0
```
**Resultado**: `z = 9.0`

**Nota**: Respeta la asociatividad izquierda: `(y + x) - 3`

---

#### **Bloque 4**: `w = (z * 2) + (y / x)`
```
t7 = 2.0       # t7 ← 2.0
t8 = z * t7    # t8 ← 9.0 * 2.0 = 18.0    [subexpresión izquierda]
t9 = y / x     # t9 ← 8.0 / 4.0 = 2.0     [subexpresión derecha]
t10 = t8 + t9  # t10 ← 18.0 + 2.0 = 20.0  [suma final]
w = t10        # w ← 20.0
```
**Resultado**: `w = 20.0`

---

#### **Bloque 5**: Impresiones
```
print x       # Imprime: 4.0
print w       # Imprime: 20.0
```

---

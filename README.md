# resumen-x64

## GDB

El manual completo esta en `info gdb`

### Comandos

- `x`: Leer de memoria e imprimir.  Usa el formato `/NFU`
  - `N`: Cantidad
  - `F`: [Formato](#formato)
  - `U`: [Unidad](#unidad)
- `p`: Evalua expresión. Usa el formato `/F`
  - `F`: [Formato](#formato)
- `c`: Continue
- `si`: "step instruction"
- `b`: Setear breakpoint
- `delete breakpoints <indice>`
- `!<sh>` ejecuta el comando `<sh>` en la terminal
  - `!make` ejecuta `make`
  - `!make clean` ejecuta `make clean`

### Dashboard

Resumen de funciones utiles:
 - `dashboard <modulo>`: Habilita/deshabilita el modulo
 - `dashboard expressions`
   - `dashboard expressions watch /16 <expr>` 
   - `dashboard expressions unwatch <indice>` 

### Formato

De la sección 10.5 "Output formats" en `info gdb`

- `x`: Hexadecimal
- `z`: Hexadecimal con padding
- `d`: Decimal
- `u`: Decimal sin signo
- `t`: Binario (*t*wo's complement)
- `a`: Dirección (Muestra un offset al label mas cercano)
- `f`: Floating point
- `s`: Null-terminated string

### Unidad

De la sección 10.6 "Examining Memory"

- `b`: De a un byte
- `h`: De a 2 bytes
- `w`: De a 4 bytes
- `g`: De a 8 bytes
  
**Nota**: El valor por defecto es el último usado

## SIMD

Los registros `xmm0` a `xmm15` se usan para las instrucciones Single Instruction Multiple Data y miden 16 bytes / 128 bits

### Nomentclatura

#### Tamaños
- `b`: byte
- `w`: word (2 bytes)
- `d`: double word, dword (4 bytes)
- `q`: quad word, qword (8 bytes)
- `dq`: double quad word (16 bytes) [^double-quad-word-size]
- `ss`: [scalar](#valores-en-registros-simd) single precision, single (float 32 bits)
- `sd`: [scalar](#valores-en-registros-simd) double precision, double (float 64 bits)

[^double-quad-word-size]: 128 bits, todo un registro XMM

#### Valores en registros SIMD
- Scalar: Un valor en el registro, los bits no usados en 0
- Packed: Varios valores en un registro

#### Alineamiento
- `a`: Aligned. Alineamiento a 16 bytes [^alineamiento-hint]
- `u`: Unaligned

[^alineamiento-hint]: El byte de las unidades es cero, ej: `0x1230` 

### Instrucciones

#### Movimiento

##### Load/Store

Estas instrucciones pueden cargar y guardar desde/hacia la memoria

| Tamaño                               | Aligned | Unaligned | Scalar |
| ------------------------------------ | ------- | --------- | ------ |
| double qword[^double-quad-word-size] | MOVDQA  | MOVDQU    |        |
| single                               | MOVAPS  | MOVUPS    | MOVSS  |
| double                               | MOVAPD  | MOVUPD    | MOVSD  |
| dword                                |         |           | MOVD   |
| qword                                |         |           | MOVQ   |

##### Mover empaquetado con extensión

Estas instrucciones pueden cargar desde la memoria (load), pero **no** pueden guardar (store)

| Tamaño     | Sing extend | Zero extend |
| ---------- | ----------- | ----------- |
| byte→word  | `PMOVSXBW`  | `PMOVZXBW`  |
| byte→dword | `PMOVSXBD`  | `PMOVZXBD`  |
| byte→quad  | `PMOVSXBQ`  | `PMOVZXBQ`  |
| word→dword | `PMOVSXWD`  | `PMOVZXWD`  |
| word→quad  | `PMOVSXWQ`  | `PMOVZXWQ`  |
| dword→quad | `PMOVSXDQ`  | `PMOVZXDQ`  |


#### Conversiones

#### Operaciones aritmeticas

#### Comparaciones

### Snippets

#### Complemento a dos:

```asm
neg: ; Parametro y ret por xmm0
    movdqa  xmm1, xmm0
    pxor    xmm0, xmm0  ; xmm0 := 0
    psub⍰   xmm0, xmm1  ; xmm0 := 0 - xmm1
    ret
```
`⍰`: [Tamaño](#tamaños) `b`, `w`, `d`, `q`

#### Complemento a uno (negación)

```asm
neg: ; Parametro y ret por xmm0
    pcmpeqd xmm1, xmm1  ; xmm1 := (-1) (o 0b1111...)
    pxor    xmm0, xmm1  ; xmm0 := xmm0 ⊻ 0b1111...
    ret
```

## C

### Convencion de C

## Assembly

### Declaración de constantes

Cuidado con la endianness:
```asm
Good_Black_RGBA: db 0, 0, 0, 0xFF
Bad_Black_RGBA: dw 0x000000FF
```
> `> x/4b &Good_Black_RGBA`
> 
> `0x..0 <Good_Black_RGBA>: 0, 0, 0, 0xFF`

El canal alfa está al máximo, el resto en 0

> `> x/4b &Bad_Black_RGBA`
> 
> `0x..0 <Bad_Black_RGBA>: 0xFF, 0, 0, 0`

El canal **rojo** está al máximo, el alfa en 0!
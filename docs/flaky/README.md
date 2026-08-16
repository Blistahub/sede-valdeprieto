# Fallos intermitentes

`docs/testing.md` §10. Cuando un test falla de forma intermitente:

1. **No** se reintenta hasta que pase. **No** se marca `skip`.
2. Se aísla y se ejecuta 50 veces para medir la tasa real.
3. Se documenta aquí en `NNN-nombre.md`: síntoma, hipótesis, causa raíz encontrada,
   corrección, y tasa antes y después.
4. Se corrige la causa, no el síntoma.

Un test desactivado para que pase CI es un fallo de proyecto, no un atajo
(`CLAUDE.md` §1, invariante III).

## Por qué este directorio importa

Dos o tres casos bien analizados aquí valen más en una entrevista técnica que doscientos
tests verdes. Un test verde demuestra que algo funcionó una vez. Una ficha de fallo
intermitente con su causa raíz demuestra que quien la escribió sabe depurar concurrencia,
esperas y estado compartido.

## Plantilla

```markdown
# NNN · Nombre del test

**Ruta:** tests/e2e/specs/…
**Detectado:** AAAA-MM-DD
**Tasa antes:** N/50
**Tasa después:** N/50
**Estado:** abierto | corregido

## Síntoma
Qué se observa, con la línea de error exacta.

## Hipótesis descartadas
Qué se pensó primero y por qué no era.

## Causa raíz
La condición observable que no se estaba usando.

## Corrección
Qué se cambió, y por qué corrige la causa y no el síntoma.
```

Ninguna ficha se cierra sin la tasa medida después de la corrección. «Ya no lo he vuelto
a ver» no es una tasa.

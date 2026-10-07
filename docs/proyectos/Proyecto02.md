# UT02 - Práctica 2: Calculadora de salario

**Alumno:** Alejandro Rodriguez
**Curso:** 2º DAW
**Asignatura:** Diseño de Interfaces Web

---

## 1. Enunciado

Hay que hacer una aplicación web de dos páginas:

1. **Primera página:** un formulario que pida:
   - **Sueldo:** un número entero mayor de 1000.
   - **Puesto:** un desplegable con tres opciones (base, directivo, alto cargo).
2. **Segunda página:** recibe los datos del formulario y calcula el sueldo final aplicando un aumento según el puesto:

| Puesto     | Aumento |
|------------|---------|
| Base       | 10 %    |
| Directivo  | 15 %    |
| Alto cargo | 20 %    |

Y tiene que mostrar por pantalla algo como esto:

```
El sueldo base es de 1200€
El complemento es del 10%
El sueldo final es de 1320€
```

---

## 2. Archivos de la práctica

| Archivo        | Para qué sirve                                          |
|----------------|---------------------------------------------------------|
| `ut02p02.html` | Página con el formulario                                |
| `ut02p02.php`  | Página que recibe los datos y calcula el sueldo final   |
| `ut02p02.css`  | Estilos que usan las dos páginas                        |

Los tres archivos tienen que estar en la **misma carpeta** y se prueban desde un servidor local con PHP (yo uso XAMPP), porque el PHP no se ejecuta si abres el archivo directamente en el navegador.

---

## 3. Primera página: el formulario (`ut02p02.html`)

Es un HTML normal con un `<form>` que manda los datos a `ut02p02.php` con el método **POST**.

```html
<form action="ut02p02.php" method="post">
    <label for="sueldo">Sueldo base (€):</label>
    <input type="number" id="sueldo" name="sueldo" min="1001" step="1" required>

    <label for="puesto">Puesto:</label>
    <select id="puesto" name="puesto" required>
        <option value="" disabled selected>-- Selecciona un puesto --</option>
        <option value="base">Base</option>
        <option value="directivo">Directivo</option>
        <option value="alto cargo">Alto cargo</option>
    </select>

    <button type="submit">Calcular</button>
</form>
```

Lo más importante:

- `type="number"` hace que solo se puedan escribir números.
- `min="1001"` y `step="1"` hacen que tenga que ser un **entero mayor de 1000**.
- `required` obliga a rellenar el campo antes de enviar.
- El `<select>` tiene una primera opción vacía (`disabled selected`) para que el usuario tenga que elegir un puesto de forma consciente.
- Uso `method="post"` para que los datos no salgan en la URL.

### Captura: formulario vacío

![Formulario vacío](/docs/img/01_formulario.png)

### Captura: formulario relleno

![Formulario relleno](/docs/img/02_formulario_relleno.png)

### Captura: validación del navegador

Si escribo un sueldo que no es mayor de 1000, el navegador avisa y no deja enviar el formulario gracias al atributo `min`.

![Validación del navegador](/docs/img/03_validacion_navegador.png)

---

## 4. Segunda página: el cálculo (`ut02p02.php`)

Aquí está la parte de programación. Lo he dividido en pasos.

### 4.1. Comprobar que llegan datos

Si alguien entra directamente en `ut02p02.php` sin pasar por el formulario, lo mando de vuelta al formulario.

```php
if ($_SERVER["REQUEST_METHOD"] !== "POST") {
    header("Location: ut02p02.html");
    exit;
}
```

### 4.2. Recoger los datos

```php
$sueldo = $_POST["sueldo"] ?? "";
$puesto = $_POST["puesto"] ?? "";
```

`$_POST` es un array que guarda lo que se envía en el formulario. El `?? ""` pone un valor vacío si el dato no existe, así no sale un error.

### 4.3. Validar en el servidor

Aunque el formulario ya valida en el navegador, **no me puedo fiar solo de eso**, porque alguien puede saltárselo. Por eso vuelvo a comprobarlo en PHP:

```php
if (filter_var($sueldo, FILTER_VALIDATE_INT) === false || (int)$sueldo <= 1000) {
    $error = "El sueldo debe ser un número entero mayor de 1000.";
}
```

### 4.4. Elegir el porcentaje según el puesto

Uso un `switch`, que es lo más cómodo cuando hay varias opciones posibles:

```php
switch ($puesto) {
    case "base":
        $porcentaje = 10;
        break;
    case "directivo":
        $porcentaje = 15;
        break;
    case "alto cargo":
        $porcentaje = 20;
        break;
    default:
        $porcentaje = 0;
        $error = "El puesto seleccionado no es válido.";
}
```

### 4.5. Calcular el sueldo final

```php
$sueldoFinal = $sueldo + ($sueldo * $porcentaje / 100);
```

Por ejemplo, con 1200 € y puesto base: `1200 + (1200 * 10 / 100) = 1320`.

### 4.6. Mostrar el resultado

Mezclo HTML con PHP. Si hay error, se enseña el mensaje; si no, se enseñan los tres datos:

```php
<?php if ($error !== ""): ?>
    <p class="error"><?= htmlspecialchars($error) ?></p>
<?php else: ?>
    <p>El sueldo base es de <strong><?= $sueldo ?>€</strong></p>
    <p>El complemento es del <strong><?= $porcentaje ?>%</strong></p>
    <p>El sueldo final es de <strong><?= $sueldoFinal ?>€</strong></p>
<?php endif; ?>
```

He usado `htmlspecialchars()` en el mensaje de error como medida de seguridad básica, para que no se pueda colar código HTML.

---

## 5. Pruebas realizadas

### Puesto base (10 %) con sueldo 1200 €

![Resultado puesto base](/docs/img/04_resultado_base.png)

### Puesto directivo (15 %) con sueldo 2000 €

![Resultado puesto directivo](/docs/img/05_resultado_directivo.png)

### Puesto alto cargo (20 %) con sueldo 3000 €

![Resultado puesto alto cargo](/docs/img/06_resultado_alto_cargo.png)

### Error de validación en el servidor

Si los datos no son válidos (por ejemplo un sueldo de 900 € enviado saltándose el formulario), PHP enseña un mensaje de error en lugar del cálculo.

![Error del servidor](/docs/img/07_error_servidor.png)

### Resumen de resultados

| Sueldo | Puesto     | Aumento | Sueldo final |
|--------|------------|---------|--------------|
| 1200 € | Base       | 10 %    | 1320 €       |
| 2000 € | Directivo  | 15 %    | 2300 €       |
| 3000 € | Alto cargo | 20 %    | 3600 €       |

---

## 6. Estilos (`ut02p02.css`)

Para que no se vea feo he hecho una hoja de estilos sencilla que comparten las dos páginas:

- El contenido va en una "tarjeta" blanca centrada (`max-width` y `margin: 40px auto`).
- Los campos del formulario ocupan todo el ancho (`width: 100%`) y uso `box-sizing: border-box` para que no se salgan de la tarjeta.
- El botón tiene un color azul y cambia un poco al pasar el ratón por encima (`:hover`).
- Los errores salen en rojo con la clase `.error`.

---

## 7. Conclusiones

- He aprendido a enviar datos de un formulario a otra página con **POST** y a recogerlos en PHP con `$_POST`.
- He visto que hay que validar **tanto en el navegador como en el servidor**.
- He practicado el `switch` y mezclar HTML con PHP.
- Lo más importante para que funcione es tener un servidor local encendido y abrir la página desde `localhost`, no desde el archivo.

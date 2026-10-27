# Fórmulas del vídeo: App de incidencias de mantenimiento

Listado de las fórmulas más complejas utilizadas en el vídeo.

> Las fórmulas están en formato inglés (separador de argumentos `,` y fórmulas encadenadas con `;`). Si tu Power Apps está en español, cambia `,` por `;` y `;` por `;;`.

## Pantalla principal (scrPrincipal)

Color de la barrita según el estado de la incidencia:

```
recEstado:
  Fill:
    Switch(ThisItem.Estado.Value,
          "Abierta", RGBA(209, 52, 56, 1),
          "En curso", RGBA(247, 99, 12, 1),
          RGBA(16, 124, 16, 1))
```

Fecha y prioridad en una sola línea:

```
lblFecha:
  Text:
    Text(ThisItem.'Fecha de alta', "dd/mm/yyyy") & "  ·  Prioridad " & ThisItem.Prioridad.Value
```

Buscador de incidencias (en la entrada de texto moderna se usa `.Value`, no `.Text`):

```
galIncidencias:
  Items:
    Search('Incidencias de mantenimiento', txtBuscar.Text, Title)
```

Mensaje cuando la búsqueda no encuentra nada:

```
lblSinResultados:
  Visible:
    CountRows(galIncidencias.AllItems) = 0
```

Abrir una incidencia para editarla o crear una nueva con el mismo formulario:

```
galIncidencias:
  OnSelect:
    EditForm(frmIncidencia);
    Navigate(scrFormulario, ScreenTransition.Cover)

btnNueva:
  OnSelect:
    NewForm(frmIncidencia);
    Navigate(scrFormulario, ScreenTransition.Cover)
```

## Pantalla del formulario (scrFormulario)

Título que cambia según si creas o editas:

```
lblTitulo:
  Text:
    If(frmIncidencia.Mode = FormMode.New,
       "Nueva incidencia",
       "Editar incidencia")
```

Volver atrás sin guardar:

```
icoVolver:
  OnSelect:
    ResetForm(frmIncidencia);
    Back()
```

Borrar la incidencia (el icono solo se ve al editar):

```
icoBorrar:
  Visible:
    frmIncidencia.Mode = FormMode.Edit
  OnSelect:
    Remove('Incidencias de mantenimiento', galIncidencias.Selected);
    Back()
```

Fecha de alta automática (tarjeta desbloqueada y oculta con `Visible: false`):

```
Tarjeta Fecha de alta:
  Update:
    If(frmIncidencia.Mode = FormMode.New,
       Today(),
       ThisItem.'Fecha de alta')
```

Estado: las incidencias nuevas entran siempre como "Abierta" (desplegable dentro de la tarjeta desbloqueada):

```
Desplegable de Estado:
  DefaultSelectedItems:
    If(frmIncidencia.Mode = FormMode.New,
       Filter(Choices([@'Incidencias de mantenimiento'].Estado), Value = "Abierta"),
       Filter(Choices([@'Incidencias de mantenimiento'].Estado), Value = ThisItem.Estado.Value))

Tarjeta Estado:
  Visible:
    frmIncidencia.Mode = FormMode.Edit
```

## Guardar

```
btnGuardar:
  OnSelect:
    SubmitForm(frmIncidencia)

frmIncidencia:
  OnSuccess:
    If(frmIncidencia.LastSubmit.'Fecha de alta' = Blank(),
       Patch(Incidencias_de_mantenimiento, frmIncidencia.LastSubmit, {'Fecha de alta': Today()}));
    Notify("Incidencia guardada", NotificationType.Success);
    Back()
  OnFailure:
    Notify("No se ha podido guardar: " & frmIncidencia.Error, NotificationType.Error)
```

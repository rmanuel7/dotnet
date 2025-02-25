Para pedirme una estructura clara y precisa, puedes describirla de la siguiente manera:  

### **Ejemplo de solicitud bien estructurada:**  
> Quiero un diseño en ASCII basado en CSS Grid con la siguiente estructura:  
>
> - **Grid principal**:  
>   - `grid-template-areas`:  
>     ```
>     "icon head"  
>     "nav main"
>     "nav main"  
>     ```
> - **Dentro de `main`** (div):
>   - `page-content-container`:
>     ```
>     height: 100%;
>     ```
> - **Dentro de `div`** (grid):  
>   - `grid-template-areas`:  
>     ```
>     "page-header"  
>     "main"  
>     "footer"
>     ```
> - **Dentro de `main`** (div):
>   - `summary-details-container`:
>     ```
>     height: 100%;
>     ```
> - **Dentro de `div`** (grid):  
>   - `grid-template-areas`:  
>     ```
>     "panel1 panel2"
>     ```
> - **Dentro de `panel1`** (div):  
>   - `summary-container`:  
>     ```
>     height: 100%;
>     min-width: 100%;
>     ```
> - **Dentro de `panel2`** (grid):  
>   - `grid-template-areas`:  
>     ```
>     "head"  
>     "main"  
>     ```
> - **Dentro de `head` del panel2** (grid):  
>   - `grid-template-areas`:  
>     ```
>     "title actions"   
>     ```
> - Quiero que `main` sea más ancho que `nav` y que la jerarquía se vea clara.

```
┌───────────────────────────────────────────────────────────────┐
│ ICON      │                      HEADER                       │
├───────────┼───────────────────────────────────────────────────┤
│ NAV       │                   PAGE CONTENT                    │
│           │ ┌───────────────────────────────────────────────┐ │
│           │ │                  PAGE HEADER                  │ │
│           │ ├───────────────────────────────────────────────┤ │
│           │ │           SUMMARY-DETAILS CONTAINER           │ │
│           │ │ ┌─────────────────────┬─────────────────────┐ │ │
│           │ │ │       PANEL 1       │       PANEL 2       │ │ │
│           │ │ │ ┌─────────────────┐ │ ┌─────────────────┐ │ │ │
│           │ │ │ │     SUMMARY     │ │ │ TITLE | ACTIONS │ │ │ │
│           │ │ │ └─────────────────┘ │ ├─────────────────┤ │ │ │
│           │ │ │                     │ │       MAIN      │ │ │ │
│           │ │ │                     │ └─────────────────┘ │ │ │
│           │ │ └─────────────────────┴─────────────────────┘ │ │
│           │ └───────────────────────────────────────────────┘ │
│           │ ┌───────────────────────────────────────────────┐ │
│           │ │                     FOOTER                    │ │
│           │ └───────────────────────────────────────────────┘ │
└───────────┴───────────────────────────────────────────────────┘
```

### **Explicación de por qué esta estructura es clara:**  
- Especifica los **diferentes niveles de anidación**.  
- Usa `grid-template-areas` para que se entienda cómo debe verse.  
- Explica qué elementos deben ser más grandes o pequeños.  

Si necesitas más detalles, también puedes incluir:  
- Cuánto espacio debe ocupar cada parte (ejemplo: `nav` más estrecho).  
- Cuántas filas o columnas esperas en cada subgrid.  

Con esta estructura, puedo hacer representaciones ASCII mucho más precisas y alineadas a lo que necesitas. 🚀

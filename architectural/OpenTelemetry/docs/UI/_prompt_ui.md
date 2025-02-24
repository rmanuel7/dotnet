Para pedirme una estructura clara y precisa, puedes describirla de la siguiente manera:  

### **Ejemplo de solicitud bien estructurada:**  
> Quiero un diseño en ASCII basado en CSS Grid con la siguiente estructura:  
>
> - **Grid principal**:  
>   - `grid-template-areas`:  
>     ```
>     "icon head"  
>     "nav main"  
>     ```
> - **Dentro de `main`** (subgrid):  
>   - `grid-template-areas`:  
>     ```
>     "page-header"  
>     "second-main"  
>     "footer"
>     ```
> - **Dentro de `second-main`** (otro subgrid):  
>   - `grid-template-areas`:  
>     ```
>     "main"  
>     "foot"
>     ```
> - Quiero que `main` sea más ancho que `nav` y que la jerarquía se vea clara.  

### **Explicación de por qué esta estructura es clara:**  
- Especifica los **diferentes niveles de anidación**.  
- Usa `grid-template-areas` para que se entienda cómo debe verse.  
- Explica qué elementos deben ser más grandes o pequeños.  

Si necesitas más detalles, también puedes incluir:  
- Cuánto espacio debe ocupar cada parte (ejemplo: `nav` más estrecho).  
- Cuántas filas o columnas esperas en cada subgrid.  

Con esta estructura, puedo hacer representaciones ASCII mucho más precisas y alineadas a lo que necesitas. 🚀

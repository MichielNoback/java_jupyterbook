# Design Patterns - Structural

Structural patterns target application structure. We only deal with one of these.


## Filter

:::{admonition} Filter Pattern
:class: note
Filter pattern or Criteria pattern is a design pattern that enables developers to filter a set of objects using different criteria and chaining them in a decoupled way through logical operations. 
:::

Although plain filtering of objects has been greatly simplified with the introduction of the ste=reams API, 
this pattern still has uses for all slightly more complicated cases.


### Revisiting the Factory pattern

Building composite filter objects for 

- Probe filtering for microarray  
- Primer filtering for qPCR  

This will involve complex construction that can be abstracted away in an Abstract Factory Class, subtype of FilterFactory:

- MicroarrayProbeFilterFactory
- PcrPrimerFilterFactory

Can you implement this model as a Factory class?


## Decorator

4N180XB4iNuW8!6
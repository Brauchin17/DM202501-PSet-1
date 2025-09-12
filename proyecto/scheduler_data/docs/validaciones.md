# Validación mediante Postman

Para comprobar si realmente se estan trayendo la cantidad de datos correctos se puede usar `COUNT` en Postman para ver si coincide con la cantidad de datos que esta ingestando el pipeline.

## Ejemplo

1. Se manda la misma query que en el pipeline, pero en se agrega `select count(*)`. Ejemplo: `select count(*) from Invoice where TxnDate >= '{current_start.date()}' and TxnDate <= '{current_end.date()}' `
2. El postman devuelve la cantidad:
   <img width="904" height="88" alt="image" src="https://github.com/user-attachments/assets/671afb04-b1e6-415f-b97f-ab53771696fe" />
   
   En el caso del ejemplo se tomo 1 mes y devolvió 21
3. Se ejecuta en el pipeline y se ve cuantas filas se obtuvo
   <img width="943" height="603" alt="image" src="https://github.com/user-attachments/assets/b3d10183-6ee4-4607-921f-dacde4ad7ef4" />
   
  Se puede ver como existe 21 filas también, por lo tanto se puede comprobar que estas extrayendo la cantidad de datos correctos.

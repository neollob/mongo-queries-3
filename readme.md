### Mongo-queries-3

Cargar el fichero __social.json__

      mongoimport --uri "mongodb+srv://neollob:<password>@cluster0-..." --collection social --drop --file social.json

- Obtener los usuarios que han hecho más comentarios y los que han hecho menos de entre todas las
entradas de la bd. (tip: usar la cláusula unwind para crear un documento por cada comentario de cada
post (que inicialmente estarían en un array dentro de el post correspondiente).

    - Menos comentarios

          db.social.aggregate([
            { $unwind : "$comments" },
            {"$group" : {
              _id:{name:"$comments.by.name"}, 
              comments:{$sum:1}
            }},
            { "$sort": { 
              "name": 1, 
              "comments": 1 
            }},
            { $limit : 10 }
          ])
          
    - Más comentarios

          db.social.aggregate([
            { $unwind : "$comments" },
            {"$group" : {
              _id:{name:"$comments.by.name"}, 
              comments:{$sum:1}
            }},
            { "$sort": { 
              "name": 1, 
              "comments": -1 
            }},
            { $group : 
              {
                _id:null, name_max:{$first:"$_id"}, 
                num_max:{$first:"$comments"}, 
                name_min:{$last:"$_id"},
                num_min:{$last:"$comments"} 
              } 
            }
          ])

- Crear funcion de paginacion para mostrar los mensajes correspondientes a la pagina que le indiquemos segun los documentos por pagina que nosotros le indiquemos

      let paginar=(np,nxp)=>{
        let docs=db.social.find().sort( { index: 1 } ).skip((np-1)*nxp).limit(nxp);
        docs.forEach((miDoc)=>{print(`ID:${miDoc._id}| Indice:${miDoc.index}`)})
      }
      paginar(3,10);
        
     <sub><sup>*nota: np = numero de pagina || nxp = numero por pagina*</sub></sup>


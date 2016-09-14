### >>> Cortar modelos pela respectiva EOO <<<

#### Script criado para uma trablho específico em que seriam utilizados preferencialmente ENM. Para aquelas espécies sem modelos seriam utilizados as EOO transformadas em raster.
````{r}
library(raster)
library(dismo)
library(rgdal)

# Listando os shapefiles
shps <- list.files("./eoo/_grid/pamp",pattern="\\.shp$",full.names=TRUE)

# Carrega o shapefile do Bioma
bioma <- shapefile("./biomas/poligono/pamp.shp")

# Listando os modelos das especies
modelos <- list.files ("./modelos/pamp",pattern="\\.asc.tif$",full.names=TRUE)
nome_sp <- basename(gsub(pattern=".asc.tif",replacement="",modelos))

# Carrega o raster do bioma
r_bioma <- raster("./biomas/raster/tif/pampa.tif")

for (shp in shps){
  
  ## Checar se existe modelo
  shps_nome <- gsub(pattern=".shp",replacement="",shp)
  shps_nome <- gsub(pattern="pamp_",replacement="",shps_nome)
  shps_nome <- basename(shps_nome)
  existe <- shps_nome %in% nome_sp
  
  if (existe==FALSE){
    
    # Cria um TXT com a lista das especies que nao tem modelo (log do console)
    sink(file=paste0("./modelos_cortados/pamp/_sem_modelo_pamp.txt"),append=T)
    cat(paste( shps_nome),"\n")
    sink()
    
    # Carregar shapefile da EOO para transformar em raster
    eoo <- shapefile(shp)
    
    # Cria a coluna pixel na tabela de atributos do shapfile e preenche com 1
    eoo$pixel <- 1
    
    # Rasterizar: transforma o poligono da eoo em raster. ATENCAO para poligonos muito pequenos pode ocorrer a geracao de 
    raster vazios(apenas valores 0). Mais e usar uma abordagem do tipo "spatial join" antes de rasterizar.
    r_pol <- rasterize(eoo,r_bioma,field="pixel",fun='sum',background=NA,mask=F)
    
    # Merge com o raster do bioma, neste caso funciona como se fosse uma soma de raters com extensoes diferentes
    raster_merge <- merge(r_pol,r_bioma)
    
    # Escreve o raster
    writeRaster(raster_merge,filename= paste0("./modelos_cortados/pamp/",shps_nome),format="ascii",NAflag=-9999,overwrite=TRUE)  
  }
  
  if (existe==TRUE){
    
    # Carrega o raster modelo 
    raster <- raster(paste0("./modelos/pamp/",shps_nome,".asc.tif"))
    
    # Carrega o shapefile da EOO da especie
    eoo <- shapefile(shp)
    
    # Cortando o modelo pela extensao do bioma: vai reduzir o numero de col e row do raster
    cortado <- crop(x=raster,y=bioma)
    
    # Cortando o modelo pela mascara da EOO
    masked <- mask(cortado,mask=eoo)
    
    # Merge do raster do bioma com o raster do modelo para haver valores 0 nos pixel fora da EOO
    raster_merge <- merge(masked,r_bioma)
    
    # Escrevendo o raster 
    writeRaster(raster_merge,filename= paste0("./modelos_cortados/pamp/",shps_nome),format="ascii",NAflag=-9999,overwrite=TRUE)
  }
}
````
===

### >>>

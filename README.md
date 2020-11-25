# eleicoesuberlandia

#Carregando bibliotecas
library (tidyverse)
library(tidyr)
library (sf)
library (ggplot2)


#abrindo arquivo

votosmg <- read_delim("~/bweb_1t_MG_181120201549.csv", 
                      ";", escape_double = FALSE, locale = locale(encoding = "Latin1"), 
                      trim_ws = TRUE)

#selecionando votos para prefeitura
votosmg %>% 
  filter(NM_MUNICIPIO == "UBERLÂNDIA") %>% 
  filter(DS_CARGO_PERGUNTA == "Prefeito") %>% 
  select(NR_ZONA, NR_SECAO, CD_CARGO_PERGUNTA, DS_CARGO_PERGUNTA, NR_PARTIDO, QT_COMPARECIMENTO,
         QT_ABSTENCOES, QT_VOTOS, NR_VOTAVEL, DS_TIPO_VOTAVEL, NM_VOTAVEL, SG_PARTIDO,
         NR_LOCAL_VOTACAO, NM_MUNICIPIO) %>% 
  group_by(NM_VOTAVEL, NR_ZONA, NM_MUNICIPIO) %>% 
  summarise (votos = sum(QT_VOTOS)) %>% 
  rename(zona = NR_ZONA)->  prefeitura

#selecionando votos para vereadores
votosmg %>% 
  filter(NM_MUNICIPIO == "UBERLÂNDIA") %>% 
  filter(DS_CARGO_PERGUNTA == "Vereador") %>% 
  select(NR_ZONA, NR_SECAO, CD_CARGO_PERGUNTA, DS_CARGO_PERGUNTA, NR_PARTIDO, QT_COMPARECIMENTO,
         QT_ABSTENCOES, QT_VOTOS, NR_VOTAVEL, DS_TIPO_VOTAVEL, NM_VOTAVEL, SG_PARTIDO,
         NR_LOCAL_VOTACAO, NM_MUNICIPIO) %>% 
  group_by(NM_VOTAVEL, NR_ZONA, NM_MUNICIPIO) %>% 
  summarise (votos = sum(QT_VOTOS)) %>% 
  rename(zona = NR_ZONA)-> vereadores


#abrindo arquivo coordenadas dos locais de votação de uberlandia

#gsheet::gsheet2tbl("https://docs.google.com/spreadsheets/d/1wBE5Ouat1xxfA5YWZX78SRBOuil7nzyN0BybBgNZ9F8/edit#gid=0")-> uberlandia
#desconfigurou a longitude, importei manualmente para garntir que foi correto



#baixando e importando dados do mapa de Uberlândia


dir.create("shp", showWarnings = FALSE)

u_ibge <- paste0(
  "ftp://geoftp.ibge.gov.br/organizacao_do_territorio/",
  "malhas_territoriais/malhas_municipais/",
  "municipio_2015/UFs/MG/mg_municipios.zip", 
  collapse = "")

httr::GET(u_ibge, 
          httr::write_disk("shp/mg.zip"),
          httr::progress())

#unzip("shp/mg.zip", exdir = "shp/") #tive que fazer manualmente

dir("shp")

municipio <- st_read("shp/31MUE250GC_SIR.shp", quiet = TRUE)

#formatando bancos para juntá-los
municipio %>% 
  rename(NM_MUNICIPIO = NM_MUNICIP ) %>%  
  filter(NM_MUNICIPIO == "UBERLÂNDIA") -> mapaudi


#transformando coordenadas do planilha e Shapefile
coords <- st_as_sf(uberlandia, 
                   coords = c("lat", "long"), 
                   crs = st_crs(mapaudi), 
                   agr = "constant")


glimpse(uberlandia)

#plotando o mapa
coords%>% 
  ggplot()+
  geom_sf()
  

  

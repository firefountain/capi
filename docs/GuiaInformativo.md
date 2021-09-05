# Guia Informativo

![Capi Logo](http://10.4.0.59/capi/Documentation/Assets/img/capi.png "Capi Logo")

# Index
1. [Intro](#intro)
2. [Wordpress](#wordpress)
3. [CAPI](#capi)
	* [SIGES Model](#sigesmodel) 
	* [CAPI SIGES Model](#capisigesmodel)
	* [CAPI SIGES controller](#capisigescontroler) 

## Intro <a name="intro"></a>

> **Pressupostos**
>
>O novo Guia de Cursos vai ter como base uma plataform Wordpress com um tema desenvolvido de base( *a verificar* ) e um plugin que vai permitir a integração dos contéudos com a CAPI.
>
>O plugin terá de ter em conta que existem dados que só existem do lado do Guia. Sendo assim o Guia passa a ser o **Owner** da informação **NÃO TÉCNICA / Caracterização** de Cursos (metadados)

Segundo os pressupostos descritos acima, o Guia vai receber informação proveniente de outras fontes, agregar a mesma e disponibilizar a mesma directamente para a **CAPI**

Sendo assim, o plugin terá de:

* **Fornecer** informação *técnica* sobre Unidades Curriculares, proveniente do **SIGES** ao Wordpress
* **Fornecer** informação *técnica* sobre Cursos, proveniente do **SIGES** ao Wordpress
* **Fornecer** informação de *metadados* sobre Cursos, há CAPI.

A CAPI, terá toda a informação sobre os cursos agregada e actulizada em (temos de definir uma temporização para sincronizar com o SIGES). Sendo que a informação de metadados proveninente do Wordpress pode ser actualizada em RT.



## Wordpress <a name="wordpress"></a>
### Informação que será gerida directamente no Wordpress

A informação resultante da gestão de cursos e caracterização dos mesmos está guardada em custom post type metadada.

Informação:

* Curso
	* Grau 
	* LínguaEnsino
	* CoordenadorCurso
	* 2CoordenadorCurso
	* Vice-CoordenadorCurso
	* 2Vice-CoordenadorCurso
	* 3Vice-CoordenadorCurso
	* NotaSobreCoordenação
	* CondicoesAcesso
	* Regime de ensino
	* Acesso a Estudos mais Avançados
	* EstruturaCurricularCréditos (Maior/Minor)
	* Saídas Profissionais
	* TítuloGuia
	* PublicacaoGuia
	* LinkGuia
	* PublicaDespacho
	* NomeDespacho
	* LinkDespacho
	* PublicaBoletim
	* NomeBoletim
	* LinkBoletim
	* ObservaçõesBoletim
	* TituloRegimeTransição:
	* PublicacaoRegimeTransicao
	* Link da Transição
	* Propinas
	* PublicaCartaz
	* LinkCartaz
	* PublicaLista
	* LinkLista
	* TituloRegulamentoCurso
	* PublicacaoRegulamento
	* Link do regulemento
	* VídeoCurso1
 	* PublicacaoVideo1
	* DenominacaoVídeo 1
	* LinkVideo1
	* PublicacaoVideo2
	* DenominacaoVídeo2
	* LinkVideo2 
* Unidade Curricular
	* Horas de Trabalho
	* Horas de Contacto
	* Língua de Ensino
	* Docentes1
	* Docentes2
	* Docentes3
	* Docentes4
	* Docentes5
	* Nível
	* Sinopse
	* Competencias
	* Conteudos
	* Bibliografia
	* MetodologiasEnsino
	* Avaliação
	* Observações
	* Requisitos
	* Contactos
	* Palavras-Chave	 

	
Esta informação é gérida através do backoffice existente na instalação de wordpress do Guia Informativo e actualizada na CAPI cada vez que a mesma é alterada no wordpress.

Desta maneira a CAPI, tem sempre toda a informação agregada actualizada e disponível para ser consumidada por entidades terceiras.

![Capi Guia Informativo](http://10.4.0.59/capi/Documentation/Assets/img/capi-GI.png "Capi Guia informativo")

Do lado da CAPI a informação é agregada por leitura da DB do SIGES que tem a informação sobre UCs e Cursos e consulidada com informação proveninente do GI (*Guia Informativo*)

##CAPI <a name="capi"></a>
Como é possível de verificar no gráfico abaixo, a ligação com o **SIGES** é efectuado através da **Base de Dados Oracle**

![Capi-Guia-Informativo-SIGES](http://10.4.0.59/capi/Documentation/Assets/img/capi-GI-siges.png "Capi Guia informativo SIGES")

Para tal acontecer a CAPI (em CodeIgniter4) vai ter de ter várias DB connectors associados a um controlado especifico. 

Desse modo, o controlados especifico para os dados provenientes do **SIGES** tem uma ligação distinta para a DB oracle do **SIGES** (*verificar acesso? tunnel SSH? keys?*) 

Exemplo de ficheiro .env com definição de duas DBs

```json
# CAPI Database
database.default.hostname = localhost
database.default.database = capi
database.default.username = root
database.default.password = root
database.default.DBDriver = MySQLi

# SIGES Database
database.siges.hostname = '192.168.0.109:1521/orcl'
database.siges.database = siges
database.siges.username = admin
database.siges.password = Admin@123
database.siges.DBDriver = oci8
```

Exemplo de ficheiro /config/Database.php com definição de ligação a SIGES

```json
    /**
     * The SIGES database connection.
     *
     * @var array
     */
    public $siges = [
        'DSN'      => '',
        'hostname' => '192.168.0.109:1521/orcl',
        'username' => '',
        'password' => '',
        'database' => '',
        'DBDriver' => 'oci8',
        'DBPrefix' => '',
        'pConnect' => false,
        'DBDebug'  => (ENVIRONMENT !== 'production'),
        'charset'  => 'utf8',
        'DBCollat' => 'utf8_general_ci',
        'swapPre'  => '',
        'encrypt'  => false,
        'compress' => false,
        'strictOn' => false,
        'failover' => [],
        'port'     => 3306,
    ];
```

#### Siges model <a name="sigesmodel"></a>

Por último temos o nosso modelo que vai permitir fazer *queries* há DB de Oracle que tem os dados de Cursos e respectivas UCs

```json
<?php
namespace App\Models;
use CodeIgniter\Model;
class Siges extends Model
{
	protected $DBGroup              = 'siges';
	protected $table                = 'siges';
	protected $primaryKey           = 'id';
	protected $useAutoIncrement     = true;
	protected $insertID             = 0;
	protected $returnType           = 'array';
	protected $useSoftDeletes       = false;
	protected $protectFields        = true;
	protected $allowedFields        = [];

	// Dates
	protected $useTimestamps        = false;
	protected $dateFormat           = 'datetime';
	protected $createdField         = 'created_at';
	protected $updatedField         = 'updated_at';
	protected $deletedField         = 'deleted_at';

	// Validation
	protected $validationRules      = [];
	protected $validationMessages   = [];
	protected $skipValidation       = false;
	protected $cleanValidationRules = true;

	// Callbacks
	protected $allowCallbacks       = true;
	protected $beforeInsert         = [];
	protected $afterInsert          = [];
	protected $beforeUpdate         = [];
	protected $afterUpdate          = [];
	protected $beforeFind           = [];
	protected $afterFind            = [];
	protected $beforeDelete         = [];
	protected $afterDelete          = [];
}
```
> Atenção que este definição de tabela é apenas um exemplo que permite montar CRUD rápidamente - **não será usado no caso do modelo de siges**


#####Operações necessárias já identificadas para o model SIGES: 

* **GET**:
	* **getUCs** - model que irá pingar a DB do SIGES e colectar toda a informação referente a UCs
	* **getCursos** - model que irá pingar a DB do SIGES e colectar toda a informação referente a CURSOS
	
> Para ambos os métodos já foram identificadas tabelas para recolher dados ver [EXCEL](https://uabpt-my.sharepoint.com/:x:/r/personal/vrodrigues_uab_pt/Documents/_Servi%C3%A7os%20de%20Inform%C3%A1tica/Projetos/GuiaInformativo/Documenta%C3%A7%C3%A3o/levantamento%20e%20mapeamento%20da%20informa%C3%A7%C3%A3o%20GuiaInformat%C3%ADvo-SIGES.xlsx?d=wba031c4b3665480f8cf913dc85d04b68&csf=1&web=1&e=mh50Z3)	


#### CAPI Siges model <a name="capisigesmodel"></a>

Ainda temos o model que permite guardar a informação que referente a **UCS** e **CURSOS** na **CAPI**

Este model é responsável por guardar/ler esta informação de modo a permitir ao controlador disponibilizar a informação a entidades terceiras.

```json
class CapiSiges extends Model
{
	protected $DBGroup              = 'default';
	protected $table                = 'capisiges';
	protected $primaryKey           = 'id';
	protected $useAutoIncrement     = true;
	protected $insertID             = 0;
	protected $returnType           = 'array';
	protected $useSoftDeletes       = false;
	protected $protectFields        = true;
	protected $allowedFields        = [];

	// Dates
	protected $useTimestamps        = false;
	protected $dateFormat           = 'datetime';
	protected $createdField         = 'created_at';
	protected $updatedField         = 'updated_at';
	protected $deletedField         = 'deleted_at';

	// Validation
	protected $validationRules      = [];
	protected $validationMessages   = [];
	protected $skipValidation       = false;
	protected $cleanValidationRules = true;

	// Callbacks
	protected $allowCallbacks       = true;
	protected $beforeInsert         = [];
	protected $afterInsert          = [];
	protected $beforeUpdate         = [];
	protected $afterUpdate          = [];
	protected $beforeFind           = [];
	protected $afterFind            = [];
	protected $beforeDelete         = [];
	protected $afterDelete          = [];
}
```
> Atenção que este definição de tabela é apenas um exemplo que permite montar CRUD rápidamente - **não será usado no caso do modelo de CAPISiges**

Neste momento já temos a base criada para poder criar um controlador que possa interagir com o SIGES e guardar os dados do lado do CAPI.

#### CAPI Siges controler <a name="capisigescontroler"></a>

```json
<?php

namespace App\Controllers;

use App\Controllers\BaseController;
use App\Models\Siges;
use App\Models\capiSiges;

class Siges extends BaseController
{
	public function index()
	{
		//
	}
	
}
```
> De salientar a utilização de ambos os models

####Métodos necessárias já identificadas para o controlador SIGES:
* **syncUCs** - método que irá recolher informação de **UCs** ao model [**SIGES**](#sigesmodel)  e transpor informação para o model [**capiSIGES**](#sigesmodel) 
* **syncCursos** - método que irá recolher informação de **Cursos** ao model [**SIGES**](#sigesmodel) e transpor informação para o model [**capiSIGES**](#sigesmodel)


![Capi-GI-arc](http://10.4.0.59/capi/Documentation/Assets/img/capi-GI-siges-arc.png "CAPI Guia informativo SIGES ARC")
 


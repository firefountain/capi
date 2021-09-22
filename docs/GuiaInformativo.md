

![Capi Logo](http://10.4.0.59/capi/Documentation/Assets/img/capi.png "Capi Logo")

# Guia Informativo

# Index
1. [Intro](#intro)
2. [Wordpress](#wordpress)
    * [Plugin Wordpress](#wpplugin)
    * [CRON](#wpcron)
3. [CAPI](#capi)
	* [SIGES Model](#sigesmodel) 
	* [CAPI SIGES Model](#capisigesmodel)
	* [CAPI SIGES controller](#capisigescontroler)
	* [CAPI GI API](#capigi) 
		* [CAPI GI Controller](#capigicontroller)  

## Intro <a name="intro"></a>

> **Pressupostos**
>
>O novo Guia Informativo vai ter como base uma plataform Wordpress com um tema desenvolvido de base( *a verificar* ) e um plugin que vai permitir a integração dos contéudos com a CAPI.
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

### Plugin Wordpress <a name="wpplugin"></a>
Esta informação é gérida através do backoffice existente na instalação de wordpress do Guia Informativo e actualizada na CAPI cada vez que a mesma é alterada no wordpress.

![Capi WP](http://10.4.0.59/capi/Documentation/Assets/img/capi-wp3.png "Capi wp")

O plugin que tem de ser desenvolvido vai disponibilizar essa comunicação através da utilização de *hook* do wordpress *publish_post*. Desta maneira quando é actualizada informação no wordpress, é chamado um ponto na api que irá actualizar essa informação na DB do CAPI.

Desta maneira a CAPI, tem sempre toda a informação agregada actualizada e disponível para ser consumidada por entidades terceiras.

![Capi WP](http://10.4.0.59/capi/Documentation/Assets/img/capi-wp2.png "Capi wp")

### CRON <a name="wpcron"></a>
Para sincronizar informação novamente com o Guia Informativo, o plugin vais disponibilizar um processo CRON. 

Este processo CRON, vai utilizar os pontos **getUCs** e **getCursos** para sincronizar informação já existente no Wordpress usando como chaves primárias **cd_UC** e **cd_Curso** e tem de ter em conta a **data de actualização**

O processo CRON não pode usar os wp_id porque os mesmo não são estáticos e no processo de sincronização os mesmo vão ser alterados. Ou seja, o processo de sincronização tem de usar como chave o **cd_UC** e **cd_Curso**. Desta maneira não existe a necessidade de guardar os wp_ids.


No processo de sincronização, o plugin tem de **apagar toda a informação que existe no WP** e recriar essa informação com a informação proveniente da CAPI.


Exemplo:

```json
// Remove all ancient info
$resultP = $wpdb->query($wpdb->prepare("DELETE p, pm
                      FROM wp_posts p
                      JOIN wp_postmeta pm
                      ON pm.post_id = p.id
    WHERE p.post_content = %d AND p.post_type = %s", $cd_curso, "cursos"));
```

Após a limpeza da informação já existente, a nova informação é carregada.

Exemplo:

```json
$cursoDetails = array(
  'post_title' => $curso->title,
  'post_name' => $curso->title,
  'post_status' => 'publish',
  'post_type' => 'cursos',
  'post_content' => $curso,
  'post_date' => $curso->year . '-01' . '-01',
);
$publicationID = wp_insert_post($publicationDetails, true);
```

**ATENÇÃO**: Todo este processo de sincronização tem de ter em conta **apenas** Cursos/UCs que foram alterados. A CAPI tem de permitir a *filtragem* de informação de Cursos/UCs por data de alteração. 
Apenas os Cursos/UCs que foram alterados são realmente removidos da DB. Desta maneira não existe uma necessidade de fazer updates a toda a informação.


Este processo pode correr todos os dias ou várias vezes ao dia.



![Capi Guia Informativo](http://10.4.0.59/capi/Documentation/Assets/img/capi-GI.png "Capi Guia informativo")

Do lado da CAPI a informação é agregada por leitura da DB do SIGES que tem a informação sobre UCs e Cursos e consulidada com informação proveninente do GI (*Guia Informativo*)

## CAPI <a name="capi"></a>
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


##### Operações necessárias já identificadas para o model SIGES: 

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

#### Métodos necessárias já identificadas para o controlador SIGES:
* **syncUCs** - método que irá recolher informação de **UCs** ao model [**SIGES**](#sigesmodel)  e transpor informação para o model [**capiSIGES**](#sigesmodel) 
* **syncCursos** - método que irá recolher informação de **Cursos** ao model [**SIGES**](#sigesmodel) e transpor informação para o model [**capiSIGES**](#sigesmodel)


![Capi-GI-arc](http://10.4.0.59/capi/Documentation/Assets/img/capi-GI-siges-arc.png "CAPI Guia informativo SIGES ARC")

Com este controlador podemos agregar e consolidar a informação e preparar a mesma para consumo


## API e Consumo <a name="capigi"></a>

Para que esta informação possa ser consumida por terceiros é necessário implementar um novo controller que terá os verbos e respectivos métodos. 
Este mesmo controller terá também de ter um novo model que recupera a informação da **CAPI DB**

#### CAPI GI controler <a name="capigicontroller"></a>

Este controller é responsável por centralizar a informação que poderá ser consumida por terceiros, como o Guia Informativo. 

##### Métodos necessárias já identificadas para o controlador CAPIGI:
* **GET**
	* **getUCs** - método que irá listar toda a informação referente a todas as **UC** que existem disponíveis em [**CAPI GI**](#capigimodel)
	* **getUC** - método que irá listar toda a informação referente a **uma única UC** que existe disponível em [**CAPI GI**](#capigimodel)
	* **getCursos** - método que irá recolher informação de todos os **Cursos** que existem disponíveis em [**CAPI GI**](#capigimodel)
	* **getCurso** - método que irá recolher informação referente a **um único CURSO** que existe disponível em [**CAPI GI**](#capigimodel)
* **PUT**
	* **updateUC** - método que permite actualizar a informação descritiva (*metadados*) de uma **determinada UC**
	* **updateCurso** - método que permite actualizar a informação descritiva (*metadados*) de uma **determinado CURSO**

#### CAPI GI model <a name="capigimodel"></a>

Para que a informação possa ser disponibilizada pelo controller é necessária a implementação de um model para gestão da informação na **CAPI DB**

##### Métodos necessárias já identificadas para o model CAPIGI:
* **getUCs** - método que irá selecionar IDs/nomes (*será que é necessário mais alguma coisa?*) de **UCs** para possibilitar consulta posteriormente
* **getUC** - método que irá selecionar toda a informação referente a **uma única UC** 
* **getCursos** - método que irá selecionar IDs/nomes (*será que é necessário mais alguma coisa?*) de **CURSOS** para possibilitar consulta posteriormente
* **getCurso** - método que irá selecionar toda a informação referente a **um únicao CURSO** 
* **updateUC** - método que permite actualizar a informação descritiva (*metadados*) de uma **determinada UC**
* **updateCurso** - método que permite actualizar a informação descritiva (*metadados*) de um **determinado CURSO**

 
![Capi-GI-arc2](http://10.4.0.59/capi/Documentation/Assets/img/capi-GI-siges-arc2.png "CAPI Guia informativo SIGES ARC")

##### Rotas para o controller

```json
/*
 * --------------------------------------------------------------------
 * Guia Informativo Routes Definitions
 * --------------------------------------------------------------------
 */

$routes->group('guia', function($routes)
{
    // GETS
    // UCs
    $routes->get('ucs', 'capigi::getUCs');
    $routes->get('ucs/(:num)', 'capigi::getUC/$1');
    // Cursos
    $routes->get('cursos', 'capigi::getCursos');
    $routes->get('cursos/(:num)', 'capigi::getCurso/$1');

    //PUTS
    $routes->put('ucs/(:num)', 'capigi::updateUC/$1');
    $routes->put('cursos/(:num)', 'capigi::updateCurso/$1');
});
```
![Capi-GI-arc3](http://10.4.0.59/capi/Documentation/Assets/img/capi-GI-siges-arc3.png "CAPI Guia informativo SIGES ARC")



---
layout: post
title:  "Descargando datos de leyes"
date:   2013-01-06
categories: web unix
---


El [sitio de la Asamblea Legislativa de Costa Rica](http://www.asamblea.go.cr) permite a cualquier persona descargar las leyes (en archivo PDF). Sin embargo, las páginas son la interfaz genérica de Microsoft Sharepoint y es cualquier cosa menos amigable (ni para personas ni para programas).

El objetivo que documento en este artículo es la descarga de la información relacionada (metadata) de todas leyes para su posterior procesamiento.

Cada ley tiene tres grupos de información asociados:

- Datos generales como número de ley, título, publicación, etc
- Proyectos de ley
- Afectaciones

Esta información se presenta en la consulta Detalles de Leyes[^1], en donde se pasa en el parámetro *Numero_Ley* la ley que se quiere consultar. En la página que nos da el sistema, los tres grupos de información antes descritos vienen dentro del cuerpo de distintos elementos `div`. Por `div` id serían:

- WebPartWPQ7: Datos generales
- WebPartWPQ8: Proyectos de ley
- WebPartWPQ5: Afectaciones

Lo que debemos hacer es implementar la extracción automática de la información de todas las leyes utilizando esa página.

Ahora, debemos recordar que la información no viene dentro de una estructura limpia en el HTML sino como parte de la sopa de tablas y divs que genera Sharepoint. Debido a esto, en esta etapa no nos vamos a preocupar por sacar la información puntual sino la estructuras principales que contienen los datos que nos interesan y más adelante implementaremos la extracción de la información.

## Solución

El proyecto [PhantomJS](http://phantomjs.org/) encapsula [WebKit](http://www.webkit.org/) con todo y el API de Javascript en forma headless, es decir, sin interfaz gráfica de usuario, permitiendo la manipulación programática del motor WebKit con soporte de los estándar de Web actuales, que es algo que necesitamos para poder interactuar con el sitio de la Asamblea Legislativa o casi que cualquier otro sitio moderno.

### Guardar datos de una ley

Lo que vamos a hacer es guardar cada nodo del HTML original (que contiene la información que nos interesa) de cada ley en un archivo por ley. Como ya tenemos los identificadores de los elementos que contienen la información que queremos extraer, basta con crear unas pocas líneas en Javascript que serán utilizadas por PhantomJS.

```javascript
var page = require('webpage').create(),
system = require('system'), address;

address = system.args[1];
page.open(address, function () {
    page.evaluate(function () {
    });

    var metadata = page.evaluate(function () {
        return document.getElementById('WebPartWPQ7').innerHTML;
    });
    var proyectos  = page.evaluate(function () {
        return document.getElementById('WebPartWPQ8').innerHTML;
    });

    var afectaciones  = page.evaluate(function () {
        return document.getElementById('WebPartWPQ5').innerHTML;
    });

    console.log("<div class='metadata'>");
    console.log(metadata);
    console.log("</div>");
    console.log("<div class='proyectos'>");
    console.log(proyectos);
    console.log("</div>");
    console.log("<div class='afectaciones'>");
    console.log(afectaciones);
    console.log("</div>");

    phantom.exit();
});
```

Para facilitar la lectura a simple vista de los archivos y la lectura del código que utilizaremos en el futuro para extraerla, cada uno de los grupos de información estará envuelto en su propio div con clases descriptivas: **metadata**, **proyectos** y **afectaciones**. Eso lo realizamos con la siguiente ejecución (ejemplo para una ley):

    phantomjs descarga-metadata-leyes.js http://www.asamblea.go.cr/Centro_de_Informacion/Consultas_SIL/Pginas/Detalle%20Leyes.aspx?Numero_Ley=3578 > 3578.ley`

Ejecutando en la terminal el archivo de Javascript anterior y redireccionando la salida al archivo **3578.ley**, vamos a tener como resultado:

```javascript
<div class='metadata'>
    <div id="bddwpDiv_ctl00_m_g_48e7fc16_716e_484a_8165_66a9bb2ca6ae">
    <table class="ms-toolbar" cellpadding="2" cellspacing="0" border="0" id="ctl00_m_g_48e7fc16_716e_484a_8165_66a9bb2ca6ae_BdwpActionBar" width="100%">
    <tbody><tr>

    <td class="ms-toolbar" nowrap="true">

    <table cellpadding="1" cellspacing="0" border="0">
    <tbody><tr>

    <td class="ms-toolbar" nowrap="" style="padding: ">
    <a href="/Centro_de_Informacion/Consultas_SIL/Pginas/#" id="ctl00_m_g_48e7fc16_716e_484a_8165_66a9bb2ca6ae_BdwpActionBar_RptControls_ctl00_LinkText" class="ms-toolbar" title="Ver texto" onclick="window.open('\u002fsil_access\u002fver_ley.aspx?Numero_Ley=3578'); return false;">Ver texto</a>
    <a id="ctl00_m_g_48e7fc16_716e_484a_8165_66a9bb2ca6ae_BdwpActionBar_RptControls_ctl00" href="#" style="visibility:hidden;"></a>
    </td>
    </tr>
    </tbody></table>

</td>

    <td class="ms-separator">|</td>

    <td class="ms-toolbar" nowrap="true">

    <table cellpadding="1" cellspacing="0" border="0">
    <tbody><tr>

    <td class="ms-toolbar" nowrap="" style="padding: ">
    <a href="/Centro_de_Informacion/Consultas_SIL/Pginas/#" id="ctl00_m_g_48e7fc16_716e_484a_8165_66a9bb2ca6ae_BdwpActionBar_RptControls_ctl01_LinkText" class="ms-toolbar" title="Ver imagen" onclick="window.open('http:\u002f\u002fwww.asamblea.go.cr:80\u002fdefault.aspx?Numero_Ley=3578'); return false;">Ver imagen</a>
    <a id="ctl00_m_g_48e7fc16_716e_484a_8165_66a9bb2ca6ae_BdwpActionBar_RptControls_ctl01" href="#" style="visibility:hidden;"></a>
    </td>
    </tr>
    </tbody></table>

</td>

    <td width="99%" class="ms-toolbar" nowrap=""><img src="/_layouts/images/blank.gif" width="1" height="18" alt=""></td>


</tr>
    </tbody></table>
    <span></span><div xmlns:asp="http://schemas.microsoft.com/ASPNET/20" xmlns:sharepoint="Microsoft.Sharepoint.WebControls" xmlns:ddwrt2="urn:frontpage:internal"></div><table border="0" width="100%" xmlns:asp="http://schemas.microsoft.com/ASPNET/20" xmlns:sharepoint="Microsoft.Sharepoint.WebControls" xmlns:ddwrt2="urn:frontpage:internal"><tbody><tr><td><table border="0" width="100%"><tbody><tr><td class="ms-descriptiontext ms-alignright"><nobr>Núm. de ley:</nobr></td><td class="ms-descriptiontext ms-alignleft" width="100%">3578</td></tr><tr><td class="ms-descriptiontext ms-alignright"><nobr>Título:</nobr></td><td class="ms-descriptiontext ms-alignleft" width="100%">LEY Nº 3578 DE 03 DE NOVIEMBRE DE 1965 ( SIN TITULO Y CONOCIDA COMO: EMPRÉSTITO DEL BANCO NACIONAL DE COSTA RICA CON EL BANCO  INTERAMERICANO DE DESARROLLO POR $4 800,000,00 PARA UN PROGRAMA  DE CRÉDITO AGROPECUARIO Y REHABILITACIÓN DE TIERRAS AFECTADAS POR ERUPCIONES DEL VOLCÁN IRAZÚ )</td></tr><tr><td class="ms-descriptiontext ms-alignright"><nobr>Publicación:</nobr></td><td class="ms-descriptiontext ms-alignleft" width="100%">11/11/1965</td></tr><tr><td class="ms-descriptiontext ms-alignright"><nobr>Núm. Gaceta:</nobr></td><td class="ms-descriptiontext ms-alignleft" width="100%">256</td></tr><tr><td class="ms-descriptiontext ms-alignright"><nobr>Núm. Alcance:</nobr></td><td class="ms-descriptiontext ms-alignleft" width="100%"> </td></tr><tr><td class="ms-descriptiontext ms-alignright"><nobr>Emisión A.L.:</nobr></td><td class="ms-descriptiontext ms-alignleft" width="100%">29/10/1965</td></tr><tr><td class="ms-descriptiontext ms-alignright"><nobr>Recepción P.E.:</nobr></td><td class="ms-descriptiontext ms-alignleft" width="100%">29/10/1965</td></tr><tr><td class="ms-descriptiontext ms-alignright"><nobr>Sanción P.E.:</nobr></td><td class="ms-descriptiontext ms-alignleft" width="100%">03/11/1965</td></tr><tr><td class="ms-descriptiontext ms-alignright"><nobr>Devuelto P.E.:</nobr></td><td class="ms-descriptiontext ms-alignleft" width="100%"></td></tr><tr><td class="ms-descriptiontext ms-alignright"><nobr>Rige:</nobr></td><td class="ms-descriptiontext ms-alignleft" width="100%">11/11/1965</td></tr></tbody></table></td></tr></tbody></table><table border="0">
    <tbody><tr>
    <td class="ms-WPBody"></td>
    </tr>
    </tbody></table></div>
</div>

<div class='proyectos'>
    <div id="bddwpDiv_ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a">
    <table class="ms-menutoolbar" cellpadding="2" cellspacing="0" border="0" id="ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar" width="100%">
    <tbody><tr>


    <td class="ms-toolbar" nowrap="true">
    <span style="display:none"><menu type="ServerMenu" id="ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_ctl00"><ie:menuitem id="ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_NoActions" type="option" iconsrc="/_layouts/images/MenuNewItem.gif" onmenuclick="return false;" text="Ninguna acción" callbackitem="true" menugroupid="2147483647"></ie:menuitem></menu></span><span title="Abrir menú"><div id="ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_Menu_t" class="ms-menubuttoninactivehover" onmouseover="MMU_PopMenuIfShowing(this);MMU_EcbTableMouseOverOut(this, true)" hoveractive="ms-menubuttonactivehover" hoverinactive="ms-menubuttoninactivehover" onclick=" MMU_Open(byid('ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_ctl00'), MMU_GetMenuFromClientId('ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_Menu'),event,false, null, 0);" foa="MMU_GetMenuFromClientId('ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_Menu')" oncontextmenu="this.click(); return false;" nowrap="nowrap"><a id="ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_Menu" href="#" onclick="javascript:return false;" style="cursor:pointer;white-space:nowrap;" onfocus="MMU_EcbLinkOnFocusBlur(byid('ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_ctl00'), this, true);" onkeydown="MMU_EcbLinkOnKeyDown(byid('ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_ctl00'), MMU_GetMenuFromClientId('ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_Menu'), event);" oncontextmenu="this.click(); return false;" menutokenvalues="MENUCLIENTID=ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_Menu,TEMPLATECLIENTID=ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_ctl00" serverclientid="ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_BdwpActionBar_RptControls_ctl00_ctl00_Menu">Acciones<img src="/_layouts/images/blank.gif" border="0" alt="Utilizar Mayús+Entrar para abrir el menú (nueva ventana)"></a><img align="absbottom" src="/_layouts/images/menudark.gif" alt=""></div></span>
    </td>

    <td width="99%" class="ms-toolbar" nowrap=""><img src="/_layouts/images/blank.gif" width="1" height="18" alt=""></td>


    <td class="ms-toolbar" nowrap="true">
    &nbsp;&nbsp;
</td>

</tr>
    </tbody></table>
    <span></span><span style="display:none"><menu type="ServerMenu" id="ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_ActionsMenuTemplate"><ie:menuitem id="ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_ctl01" type="option" onmenuclick="window.open('\u002fsil_access\u002fver_texto_base.aspx?Numero_Proyecto=\u0025Numero_Proyecto\u0025'); return false;" text="Ver texto base" callbackitem="true" menugroupid="2147483647"></ie:menuitem><ie:menuitem id="ctl00_m_g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a_ctl02" type="option" onmenuclick="window.location = '/Centro_de_Informacion/Consultas_SIL/Pginas/Detalle Proyectos de Ley.aspx?Numero_Proyecto=%Numero_Proyecto%';" text="Ver detalles" callbackitem="true" menugroupid="2147483647"></ie:menuitem></menu></span><div xmlns:asp="http://schemas.microsoft.com/ASPNET/20" xmlns:sharepoint="Microsoft.Sharepoint.WebControls"></div><table id="BdwpRows" border="0" width="100%" cellpadding="2" cellspacing="0" xmlns:asp="http://schemas.microsoft.com/ASPNET/20" xmlns:sharepoint="Microsoft.Sharepoint.WebControls"><tbody><tr valign="top"><th class="ms-vh" width="1"></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Núm. proyecto @Numero_Proyecto number;3082 ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a','dvt_sortfield={Numero_Proyecto};dvt_sortdir={' + 'ascending' + '}');">Núm. proyecto</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Asunto @Asunto_Proyecto text;3082 ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a','dvt_sortfield={Asunto_Proyecto};dvt_sortdir={' + 'ascending' + '}');">Asunto</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Tipo @Descripcion_Tipo text;3082 ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a','dvt_sortfield={Descripcion_Tipo};dvt_sortdir={' + 'ascending' + '}');">Tipo</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Fecha de iniciación @Fecha_Iniciacion datetime;3082 ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a','dvt_sortfield={Fecha_Iniciacion};dvt_sortdir={' + 'ascending' + '}');">Fecha de iniciación</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Fecha de vencimiento @Fecha_Vencimiento datetime;3082 ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a','dvt_sortfield={Fecha_Vencimiento};dvt_sortdir={' + 'ascending' + '}');">Fecha de vencimiento</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Prorrogable @Prorrogable boolean;3082 ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a','dvt_sortfield={Prorrogable};dvt_sortdir={' + 'ascending' + '}');">Prorrogable</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Numero archivado @Numero_Archivado text;3082 ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a','dvt_sortfield={Numero_Archivado};dvt_sortdir={' + 'ascending' + '}');">Numero archivado</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Fecha cuatrienal @Fecha_Cuatrienal datetime;3082 ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_8c8ce1bc_0304_4ae4_902a_b4f74a9f297a','dvt_sortfield={Fecha_Cuatrienal};dvt_sortdir={' + 'ascending' + '}');">Fecha cuatrienal</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th></tr><tr><td class="ms-vb" width="1"><span ddwrt:amkeyfield="" ddwrt:amkeyvalue="''" ddwrt:ammode="select" xmlns:ddwrt="http://schemas.microsoft.com/WebParts/v2/DataView/runtime"></span></td><td class="ms-vb" style=""><div class="ms-vb-title"><table height="100%" cellspacing="0" class="ms-unselectedtitle" onmouseover="MMU_EcbTableMouseOverOut(this, true)" hoveractive="ms-selectedtitle" hoverinactive="ms-unselectedtitle" oncontextmenu="this.click(); return false;" menuformat="ArrowOnHover" downarrowtitle="Open Menu" id="__bg01002300430073004300t" onclick="showActionMenu('Cargando...','2474',false,'SIL_Instance','ProyectoLeyRef','__bg01002300430073004300');"><tbody><tr><td class="ms-vb"><a onclick="event.cancelBubble=true" href="http://www.asamblea.go.cr/Centro_de_informacion/Consultas_SIL/_layouts/ProfileRedirect.aspx?Application=SIL_Instance&amp;Entity=ProyectoLeyRef&amp;ItemId=__bg01002300430073004300" onkeydown="actionMenuOnKeyDown('Cargando...','2474',false,'SIL_Instance','ProyectoLeyRef','__bg01002300430073004300');">2474<img src="/_layouts/images/blank.gif" border="0" alt=""></a></td><td class="ms-menuimagecell" style="visibility:hidden" id="__bg01002300430073004300ti"><img src="/_layouts/images/downarrw.gif" width="13" alt=""></td></tr></tbody></table></div><span></span></td><td class="ms-vb" style="">EMPRÉSTITO DEL BANCO NACIONAL DE COSTA RICA CON EL BANCO INTERAMERICADO DE DESARROLLO POR $4.800.000 PARA UN PROGRAMA DE CRÉDITO AGROPECUARIO DE REHABILITACIÓN DE TIERRAS AFECTADAS POR ERUPCIONES DEL VOLCÁN IRAZÚ</td><td class="ms-vb" style="">PROCEDIMIENTO PROYECTO DE LEY ORDINARIO</td><td class="ms-vb" style="">15/10/1965</td><td class="ms-vb" style=""></td><td class="ms-vb" style="">False</td><td class="ms-vb" style="">0</td><td class="ms-vb" style=""></td></tr></tbody></table><table border="0">
    <tbody><tr>
    <td class="ms-WPBody"></td>
    </tr>
    </tbody></table></div>
</div>

<div class='afectaciones'>
    <div id="bddwpDiv_ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0">
    <table class="ms-menutoolbar" cellpadding="2" cellspacing="0" border="0" id="ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar" width="100%">
    <tbody><tr>


    <td class="ms-toolbar" nowrap="true">
    <span style="display:none"><menu type="ServerMenu" id="ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_ctl00"><ie:menuitem id="ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_NoActions" type="option" iconsrc="/_layouts/images/MenuNewItem.gif" onmenuclick="return false;" text="Ninguna acción" callbackitem="true" menugroupid="2147483647"></ie:menuitem></menu></span><span title="Abrir menú"><div id="ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_Menu_t" class="ms-menubuttoninactivehover" onmouseover="MMU_PopMenuIfShowing(this);MMU_EcbTableMouseOverOut(this, true)" hoveractive="ms-menubuttonactivehover" hoverinactive="ms-menubuttoninactivehover" onclick=" MMU_Open(byid('ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_ctl00'), MMU_GetMenuFromClientId('ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_Menu'),event,false, null, 0);" foa="MMU_GetMenuFromClientId('ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_Menu')" oncontextmenu="this.click(); return false;" nowrap="nowrap"><a id="ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_Menu" href="#" onclick="javascript:return false;" style="cursor:pointer;white-space:nowrap;" onfocus="MMU_EcbLinkOnFocusBlur(byid('ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_ctl00'), this, true);" onkeydown="MMU_EcbLinkOnKeyDown(byid('ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_ctl00'), MMU_GetMenuFromClientId('ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_Menu'), event);" oncontextmenu="this.click(); return false;" menutokenvalues="MENUCLIENTID=ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_Menu,TEMPLATECLIENTID=ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_ctl00" serverclientid="ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_BdwpActionBar_RptControls_ctl00_ctl00_Menu">Acciones<img src="/_layouts/images/blank.gif" border="0" alt="Utilizar Mayús+Entrar para abrir el menú (nueva ventana)"></a><img align="absbottom" src="/_layouts/images/menudark.gif" alt=""></div></span>
    </td>

    <td width="99%" class="ms-toolbar" nowrap=""><img src="/_layouts/images/blank.gif" width="1" height="18" alt=""></td>


    <td class="ms-toolbar" nowrap="true">
    &nbsp;&nbsp;
</td>

</tr>
    </tbody></table>
    <span></span><span style="display:none"><menu type="ServerMenu" id="ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_ActionsMenuTemplate"><ie:menuitem id="ctl00_m_g_e7c5a564_639d_4946_8a9c_375f47d12bc0_ctl01" type="option" onmenuclick="window.location = '/Centro_de_Informacion/Consultas_SIL/Pginas/Detalle Leyes.aspx?Numero_Ley=%Ley_Afectada%';" text="Ver ley que afecta" callbackitem="true" menugroupid="2147483647"></ie:menuitem></menu></span><div xmlns:asp="http://schemas.microsoft.com/ASPNET/20" xmlns:sharepoint="Microsoft.Sharepoint.WebControls"></div><table id="BdwpRows" border="0" width="100%" cellpadding="2" cellspacing="0" xmlns:asp="http://schemas.microsoft.com/ASPNET/20" xmlns:sharepoint="Microsoft.Sharepoint.WebControls"><tbody><tr valign="top"><th class="ms-vh" width="1"></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Ley que afecta @Ley_Afectada number;3082 ctl00$m$g_e7c5a564_639d_4946_8a9c_375f47d12bc0')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_e7c5a564_639d_4946_8a9c_375f47d12bc0','dvt_sortfield={Ley_Afectada};dvt_sortdir={' + 'ascending' + '}');">Ley que afecta</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Artículo que afecta @Articulo_que_Afecta text;3082 ctl00$m$g_e7c5a564_639d_4946_8a9c_375f47d12bc0')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_e7c5a564_639d_4946_8a9c_375f47d12bc0','dvt_sortfield={Articulo_que_Afecta};dvt_sortdir={' + 'ascending' + '}');">Artículo que afecta</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Título de la ley que afecta @Titulo_Ley_Afectada text;3082 ctl00$m$g_e7c5a564_639d_4946_8a9c_375f47d12bc0')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_e7c5a564_639d_4946_8a9c_375f47d12bc0','dvt_sortfield={Titulo_Ley_Afectada};dvt_sortdir={' + 'ascending' + '}');">Título de la ley que afecta</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Artículo afectado @Articulo_Afectado text;3082 ctl00$m$g_e7c5a564_639d_4946_8a9c_375f47d12bc0')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_e7c5a564_639d_4946_8a9c_375f47d12bc0','dvt_sortfield={Articulo_Afectado};dvt_sortdir={' + 'ascending' + '}');">Artículo afectado</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th><th class="ms-vh" align="left"><table ctxnum="1" cellspacing="0" class="ms-unselectedtitle" onmouseover="OnMouseOverAdHocFilter(this, 'Número de ley @Numero_Ley number;3082 ctl00$m$g_e7c5a564_639d_4946_8a9c_375f47d12bc0')"><tbody><tr><td width="100%" class="ms-vb" nowrap=""><a href="javascript: __doPostBack('ctl00$m$g_e7c5a564_639d_4946_8a9c_375f47d12bc0','dvt_sortfield={Numero_Ley};dvt_sortdir={' + 'ascending' + '}');">Número de ley</a></td><td><img src="/_layouts/images/blank.gif" width="13" style="visibility: hidden" alt=""></td></tr></tbody></table></th></tr></tbody></table><table border="0">
    <tbody><tr>
    <td class="ms-WPBody">No hay elementos que mostrar.</td>
    </tr>
    </tbody></table></div>
</div>
```

### Guardar los datos de todas las leyes

Eso fue necesario para guardar datos de una ley específica, pero lo que queremos es guardar los datos de todas, una por archivo. En este momento el sistema devuelve el detalle hasta la ley #9112, así que ocupamos ejecutar esa línea en la terminal más de nueve mil veces. Trabajando en Bash, vamos a escribir un pequeño script que haga justamente eso por nosotros:

```bash
#!/bin/bash
for i in $(seq 9112)
do
    echo "phantomjs ~/descarga-metadata-leyes.js http://www.asamblea.go.cr/Centro_de_Informacion/Consultas_SIL/Pginas/Detalle%20Leyes.aspx?Numero_Ley=$i > $i.ley"
done
```
*generascript.sh*

Luego de ejecutar ese script vamos a tener en el directorio de ejecución 9112 archivos .ley, cada uno tendrá tres nodos principales div cuyos cuerpos guardan la información que corresponde a cada ley. El nombre del archivo es el número de cada ley, ejemplo: **3578.ley** es el archivo que contiene los datos para la ley #3578.


---
[^1]:    Ejemplo para ley #3578 
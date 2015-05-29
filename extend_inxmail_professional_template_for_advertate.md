# Erweiterung eines Inxmail Templates für Advertate

In diesem Hilfe lernen Sie, wie Sie ein Inxmail Template für Advertate erweitern und Testen. Sie werden lernen, wie Sie

- die Content Include Datenquelle für Advertate in Inxmail anlegen
- die Content Include XSL Transformation für die Datenquelle erstellen
- das Template `html.xml` erweitern
- die Template XSL Transformation `html.xsl` erweitern
- die Advertate Erweiterung vollständig testen können

## Der Content Include - Bindeglied zwischen Inxmail und Advertate

Inxmail ruft Anzeigen Daten über den Content Include aus Advertate ab. Advertate antwortet dabei auf den HTTP GET Request von Inxmail stets mit einem HTTP OK Response (HTTP-Statuscode 200). Der HTTP Response Body ist immer ein XML-Dokument, das entweder die Anzeigendaten oder eine Fehlermeldung enhält:

Bildanzeige:

    <?xml version="1.0" encoding="UTF-8"?>
    <placement>
      <name>Skyscraper</name>
      <vert type="ImageVert">
        <link-url>https://example.com</link-url>
        <image-url>http://placehold.it/200x400</image-url>
        <type>ImageVert</type>
      </vert>
    </placement>

Textanzeige:

    <?xml version="1.0" encoding="UTF-8"?>
    <placement>
      <name>Stellenanzeige</name>
      <vert type="TextVert">
        <link-url>https://example.com</link-url>
        <headline>Tolle Angebote</headline>
        <link-text>mehr erfahren...</link-text>
        <type>TextVert</type>
        <text-image-url>http://placehold.it/100x100</text-image-url>
        <text-as-html>Lorem Ipsum...</text-as-html>
      </vert>
    </placement>

Leere Anzeige:

    <?xml version="1.0" encoding="UTF-8"?>
    <placement>
      <vert>
        <link-url>null</link-url>
        <type>null</type>
      </vert>
    </placement>

Fehlermeldung '*Anzeigenplatz nicht gebucht*':

    <?xml version="1.0" encoding="UTF-8"?>
    <exception type="info">
      <message>&lt;span style="font-size:12px;line-height:14px;color:#000000;"&gt;Advertate Meldung:&lt;/span&gt;&lt;br /&gt;&lt;span style="font-size:12px;line-height:14px;color:#000000;"&gt;Der Anzeigenplatz &lt;b&gt;Skyscraper&lt;/b&gt; ist nicht gebucht!&lt;/span&gt;</message>
    </exception>

Fehlermeldung '*Unvollständige Anzeige*':

    <?xml version="1.0" encoding="UTF-8"?>
    <exception type="warning">
      <message>&lt;span style="font-size:12px;line-height:14px;color:#000000;"&gt;Advertate Meldung:&lt;/span&gt;&lt;br /&gt;&lt;span style="font-size:12px;line-height:14px;color:#000000;"&gt;Es fehlen noch Daten für den Anzeigenplatz: &lt;b&gt;Skyscraper&lt;/b&gt;.&lt;/span&gt;&lt;br /&gt;&lt;a href="http://www.example.com/publications/1/details/#{$week_number}/placement/1"&gt;&lt;span style="font-size:12px;line-height:14px;color:#000000;text-decoration:underline;"&gt;Anzeige in Advertate bearbeiten&lt;/span&gt;&lt;/a&gt;</message>
    </exception>

Fehlermeldung '*Anzeigenplatz nicht vorhanden*':

    <?xml version="1.0" encoding="UTF-8"?>
    <exception type="error">
      <message>&lt;span style="font-size:12px;line-height:14px;color:#000000;"&gt;Advertate Meldung:&lt;/span&gt;&lt;br /&gt;&lt;span style="font-size:12px;line-height:14px;color:#000000;"&gt;Anzeigenplatz ist in Advertate nicht vorhanden!&lt;/span&gt;</message>
    </exception>

Fehlermeldung '*Kein API Key angegeben*':

    <?xml version="1.0" encoding="UTF-8"?>
    <exception type="error">
      <message>&lt;span style="font-size:12px;line-height:14px;color:#000000;"&gt;Advertate Meldung:&lt;/span&gt;&lt;br /&gt;&lt;span style="font-size:12px;line-height:14px;color:#000000;"&gt;Es wurde kein Advertate API Key angegeben. Die Anzeigendaten können daher nicht aus Advertate abgerufen werden.&lt;/span&gt;</message>
    </exception>

Fehlermeldung '*API Key ungültig*':

    <?xml version="1.0" encoding="UTF-8"?>
    <exception type="error">
      <message>&lt;span style="font-size:12px;line-height:14px;color:#000000;"&gt;Advertate Meldung:&lt;/span&gt;&lt;br /&gt;&lt;span style="font-size:12px;line-height:14px;color:#000000;"&gt;Der Advertate API Key ist ungültig. Die Anzeigendaten können daher nicht aus Advertate abgerufen werden.&lt;/span&gt;</message>
    </exception>

## Die Content Include Datenquelle

Richten Sie den Content Include in Inxmail Professional mit den folgenden Werten ein:

- Name: Frei wählbar, der Name wird später im Template für den Content Include Aufruf im Template benötigt. In den Beispielen unten hat sie den Namen *Advertate-include*.
- Art der Datenquelle: HTTP-Abfrage
- Daten folgendermaßen in E-Mails verwenden: als Text einfügen mit vorgeriger Transformation (XML)
- URL:`https://advertate-test.inxmail.com/api/placements?publicationTitle=[$publicationTitle]&sendingDate=[$sendingDate]&placementName=[$placementName]&apiKey=hfWKy300-uWayzBuVsKeEw&mailBuildMode=[$mailBuildMode]`
- URL nur einemal pro Mailversand abfragen: angehakt

*Beachten Sie*: Der Content Include den Sie hier zum Testen des von Ihnen entwickelten Templates einrichten, unterscheidet sich von dem Content Include für Ihren Kunden im Parameter `apiKey`. Der oben angegebene API Key zeigt auf eine Advertate Test Organisation in der die unten aufgeführten Testdaten hinterlegt sind.
Möchte Ihr Kunde das Template mit seiner Advertate Organisation nutzen, so muss der Wert für den Parameter `apiKey` auf den API Key Ihres Kunden geändert werden.

## Die Content Include XSL Transformation

Da Sie die obige Datenquelle mit der Option *als Text einfügen mit vorgeriger Transformation (XML)* angelegt haben, muss bei dem späteren Aufruf des Content Includes eine XSL Transformation angegeben werden. Legen Sie dazu eine Content Include Transformation mit beliebigem Namen an. Der Name der Transformation wird später im Template für den Aufruf des Content Includes benötigt. In Beispielen unten hat sie den Namen *Advertate-transformation*. Die Transformation wandelt die oben aufgeführten XML-Dokumente in das Anzeigen-HTML.

Die von Ihnen erstellte XSL Transformation muss also mit zwei Kategorien von XML Dokumenten umgegen können; `<placement>`-Dokumenten und `<exception>`-Dokumenten. Wann genau welche Antwort von Advertate kommt, wird in dem Abschnitt *Die Testdaten* aufgeschlüsselt.

Das `type`-Attribut des `<exception>`-Elementes kann verwendet werden, um die Meldungen unterschiedlich darzustellen. Als Standard wird empfohlen den `info`-, `warning`- und `error`-Meldungen unterschiedliche Hintergrundfarben zuzuweisen. Die Inxmail Professional konformen Farbwerte sind: `info`: #E7F7B7 , `warning`: #FFFFBB und `error`: #F5E5E5. Beachten Sie hierzu auch die Beispiel-Transformation, in der das `type`-Attribut verwendet wird um die Meldung in einer passenden Farbe darzustellen.

Als Orientierung für die Erstellung Ihrer XSL Transformation kann die folgende Beispiel-Transformation genommen werden:

    <?xml version="1.0" encoding="iso-8859-1"?>
    <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
      <xsl:output method="xml" media-type="text/xml" />
      <xsl:template match="/">
        <xsl:apply-templates />
      </xsl:template>
      <xsl:template match="exception">
        <xsl:variable name="message_color">
          <xsl:choose>
            <xsl:when test="@type='warning'">
              <xsl:value-of select="'#FFFFBB'" />
            </xsl:when>
            <xsl:when test="@type='info'">
              <xsl:value-of select="'#E7F7B7'" />
            </xsl:when>
            <xsl:otherwise>
              <xsl:value-of select="'#F5E5E5'" />
            </xsl:otherwise>
          </xsl:choose>
          </xsl:variable>
        <xsl:variable name="message_style">
          line-height:14px;border-width:1px;border-style:solid;padding:1.0em;border-color:#E0E0E0;background-color:<xsl:value-of select="$message_color" />;
        </xsl:variable>
        <p>
          <xsl:attribute name="style">
            <xsl:value-of select="$message_style" />
          </xsl:attribute>
          <font size="2">
            <xsl:value-of select="message" disable-output-escaping="yes"/>
          </font>
        </p>
      </xsl:template>
      <xsl:template match="placement">
        <xsl:choose>
          <xsl:when test="(vert/type = 'TextVert')">
            <hr size="1" color="#cccccc" />
            <table cellspacing="0" cellpadding="0" border="0" width="600"><tbody><tr><td style="vertical-align:top;" valign="top" align="left" width="580">
                    <table cellspacing="0" cellpadding="0" border="0" width="580"><tbody>
                        <tr><td style="vertical-align:top;" valign="top" align="left" width="580" height="16">
                            <span style="line-height:10px;font-family:Arial,Helvetica,sans-serif;font-style:normal;font-size:12px;color:#999999;">
                              Anzeige
                            </span>
                        </td></tr>
                        <tr><td style="vertical-align:top;" height="12" width="580"></td></tr>
                        <tr><td style="vertical-align:top;" valign="top" align="left" width="580">
                          <a style="text-decoration:none;">
                            <xsl:attribute name="href">
                              <xsl:text>[%url:count; </xsl:text>
                              <xsl:value-of select="vert/link-url" />
                              <xsl:text>]</xsl:text>
                            </xsl:attribute>
                            <span style="line-height:20px;font-family:Arial,Helvetica,sans-serif;font-style:normal;font-size:17px;font-weight:bold;text-decoration:none;color:#666666;">
                              <xsl:value-of select="vert/headline" />
                            </span>
                          </a>
                        </td></tr>
                        <tr><td style="vertical-align:top;" height="10" width="580"></td></tr>
                        <tr><td style="vertical-align:top;" valign="top" align="left" width="580">
                            <table cellspacing="0" cellpadding="0" border="0" width="580"><tbody>
                              <tr>
                                <td style="vertical-align:top;" valign="top" align="left" width="580">
                                  <span style="line-height:14px;font-family:Arial,Helvetica,sans-serif;font-style:normal;font-size:12px;font-weight:normal;text-decoration:none;color:#000000;">
                                    <xsl:value-of select="vert/text-as-html" disable-output-escaping="yes"/>
                                  </span><br style="font-size:12px;" />
                                </td>
                                <xsl:if test="vert/text-image-url != ''">
                                  <td style="vertical-align:top; border-left: solid #fff 10px" valign="top">
                                    <a>
                                      <xsl:attribute name="href">
                                        <xsl:text>[%url:count; </xsl:text>
                                        <xsl:value-of select="vert/link-url" />
                                        <xsl:text>]</xsl:text>
                                      </xsl:attribute>
                                      <img width="130" border="0">
                                        <xsl:attribute name="src">
                                          <xsl:value-of select="vert/text-image-url" />
                                        </xsl:attribute>
                                      </img>
                                    </a>
                                  </td>
                                  </xsl:if>
                              </tr>
                            </tbody></table>
                        </td></tr>
                        <tr><td style="vertical-align:top;" height="10" width="580"></td></tr>
                        <tr><td style="vertical-align:top;" valign="top" align="left" width="580">
                            <table cellspacing="0" cellpadding="0" border="0" width="580"><tbody><tr>
                                  <td style="vertical-align:top;" valign="top" align="left" width="580">
                                    <span style="line-height:14px;font-family:Arial,Helvetica,sans-serif;font-style:normal;font-size:12px;font-weight:normal;text-decoration:none;color:#000000;">
                                      <a>
                                        <xsl:attribute name="href">
                                          <xsl:text>[%url:count; </xsl:text>
                                          <xsl:value-of select="vert/link-url" />
                                          <xsl:text>]</xsl:text>
                                        </xsl:attribute>
                                        <xsl:value-of select="vert/link-text" />
                                      </a>
                                </span><br style="font-size:12px;" /></td></tr>
                            </tbody></table>
                        </td></tr>
                        <tr><td style="vertical-align:top;" height="10" width="580"></td></tr></tbody></table>
            </td><td style="vertical-align:top;" width="10"></td></tr></tbody></table>
            <hr size="1" color="#cccccc" />
          </xsl:when>
          <xsl:when test="(vert/type = 'ImageVert')">
            <hr size="1" color="#cccccc" />
            <table cellspacing="0" cellpadding="0" border="0" width="580"><tbody>
              <tr><td style="vertical-align:top;" valign="top" align="left" width="580" height="16">
                <span style="line-height:10px;font-family:Arial,Helvetica,sans-serif;font-style:normal;font-size:12px;color:#999999;">
                  Anzeige
                </span>
              </td></tr>
              <tr><td style="vertical-align:top;" valign="top" align="left" width="580">
                <a>
                  <xsl:attribute name="href">
                    <xsl:text>[%url:count; </xsl:text>
                    <xsl:value-of select="vert/link-url" />
                    <xsl:text>]</xsl:text>
                  </xsl:attribute>
                  <img border="0" width="600">
                    <xsl:attribute name="src">
                      <xsl:value-of select="vert/image-url" />
                    </xsl:attribute>
                  </img>
                </a>
              </td></tr>
            </tbody></table>
            <hr size="1" color="#cccccc" />
          </xsl:when>
        </xsl:choose>
      </xsl:template>
    </xsl:stylesheet>

## Das Template

Das Template, sprich die Datei `newsletter.xml`, muss um zwei Elemente erweitert werden. Ein Element *Advertate Einstellungen* und ein Element *Advertate Anzeige*, wobei Sie den Elementen auch andere Namen geben können, wenn Sie möchten.

### Element *Advertate Einstellungen*

Das Element *Advertate Einstellungen* wird benötigt um im Content Include die Parameter `publicationTitle` und `sendingDate` zu setzen.</br> **Wichtig**: Falls Ihr Template nur für einen Newsletter benutzt wird, so sollte der Parameter `publicationTitle` fest im Aufruf des Content Includes eingetragen werden. Eine freie Wahl des Publikationstitels ist nur nötig, wenn bei Ihnen ein Template für mehrere Newsletter genutzt wird.

DTD für den Fall, dass das Element *Advertate Einstellungen* die Felder *Titel* und *Versanddatum* besitzen soll:

    <!ELEMENT Settings (advertate_pub_title,advertate_pub_date)>
    <!ATTLIST Settings
      lang CDATA "Advertate Einstellungen"
      auto-generate CDATA "true">
    <!ELEMENT advertate_pub_title (#PCDATA)>
    <!ATTLIST advertate_pub_title
      help CDATA "Newslettername wie in Advertate definiert"
      lines CDATA "1"
      lang CDATA "Titel"
      auto-generate CDATA "true">
    <!ELEMENT advertate_pub_date (#PCDATA)>
    <!ATTLIST advertate_pub_date
      help CDATA "Versanddatum in diesem Format: 2014-07-31"
      lines CDATA "1"
      lang CDATA "Versanddatum"
      auto-generate CDATA "true">

DTD für den Fall, dass der Newsletter Name fest im Content Include Aufruf eingetragen ist und das Element *Advertate Einstellungen* nur das Feld *Versanddatum* besitzt:

    <!ELEMENT Settings (advertate_pub_date)>
    <!ATTLIST Settings
      lang CDATA "Advertate Einstellungen"
      auto-generate CDATA "true">
    <!ELEMENT advertate_pub_date (#PCDATA)>
    <!ATTLIST advertate_pub_date
      help CDATA "Versanddatum in diesem Format: 2014-07-31"
      lines CDATA "1"
      lang CDATA "Versanddatum"
      auto-generate CDATA "true">

Im `<newsletter>`-Element muss dann ein leeres `<Settings>`-Element angelegt werden:

    <Settings>
      <advertate_pub_title /> <!-- Diese Zeile nur für den ersten Fall -->
      <advertate_pub_date />
    </Settings>

### Element *Advertate Anzeige*

Das Element *Advertate Anzeige* wird benötigt um im Content Include den Parameter `placementName` zu setzen und repräsentiert die tatsächliche Anzeige:

    <!ELEMENT advertate (advertate_ad_name)>
    <!ATTLIST advertate
      lang CDATA "Advertate Anzeige"
      auto-generate CDATA "true">
    <!ELEMENT advertate_ad_name (#PCDATA)>
    <!ATTLIST advertate_ad_name
      help CDATA "Anzeigenplatz wie in Advertate definiert"
      lines CDATA "1"
      lang CDATA "Name"
      auto-generate CDATA "true">

Falls Ihr Kunde nur einen Anzeigentypen hat oder Ihnen bereits die verschiedenen Anzeigentypen bekannt sind (z.B. Skyscraper, Banner, etc.), so ist es für den Redakteur angenehmer wenn er diese Elemente direkt auswählen kann. In dem Falle sollten Sie also Elemente wie folgt definieren:

    <!ELEMENT advertate_skyscraper>
    <!ATTLIST advertate
      lang CDATA "Advertate Skyscraper"
      auto-generate CDATA "true">

    <!ELEMENT advertate_banner>
    <!ATTLIST advertate
      lang CDATA "Advertate Banner"
      auto-generate CDATA "true">


## Die Template XSL Transformation

Die Template XSL Transformation, also die Datei `html.xsl`, muss den Content Include Aufruf generieren, der im erzeugten Newsletter HTML dann wie folgt aussehen muss. Die mit den Fragezeichen gekennzeichneten Werte ensprchen dann den Eingaben des Redakteurs:

    [%content-include("Advertate-include");xslt("Advertate-transformation");$publicationTitle=??????;$mailBuildMode=[%mailbuildmode];$placementName=??????;$sendingDate=??????]

## Die Testdaten

Wenn das Template und Inxmail nun so wie oben beschrieben angepasst sind, sollte das Template und die Content Include Transformation mit verschiedenen Parametern getestet werden.

Dazu existiert auf Advertate Seite eine speziell vorbereitete Test Organisation mit dem API Key `hfWKy300-uWayzBuVsKeEw`. Diese Organisation hat eine Publikation mit dem Namen `Dummy`. Die Publikation besitzt vier Anzeigenplätze: `Complete Image Ad`, `Incomplete Image Ad`, `Complete Text Ad` und `Incomplete Text Ad`. Advertate Unterscheidet zwischen zwei Anzeigentypen: *Bildanzeigen* und *Textanzeigen* (XML Struktur s.o.). Sollten in einer Anzeige auf Advertate Seite Informationen fehlen, die für den Versand dieser Anzeige im Newsletter benötigt werden, so hat diese den Status `incomplete`. Dies ist z.B. der Fall wenn die Bild-URL (`<image-url>`) einer Bildanzeige fehlt. Sind alle Daten vollständig, so hat die Anzeige den Status `complete`. Neben der Vollständigkeits-Status kann eine Anzeige nur reserviert oder bereits gebucht sein.

In Inxmail kann ein Newsletter in der Vorschau angezeigt, in einem Testversand verschickt oder in einem Echtversand verschickt werden. Dieser Versandmodus hat auch Auswirkung darauf wie die Antwort von Advertate aussieht. So sollten z.B. bei einer Vorschau des Newsletters dem Redakeur angezeigt werden, dass eine Anzeige unvollständig ist. Bei einem Echtversand hingegen sollte diese unvollständige Anzeige hingegen gar nicht verschickt werden.

Auf einem Anzeigenplatz kann entweder eine Anzeige reserviert/gebucht sein. Allerdings kann es auch vorkommen, dass es gar keinen Interessenten für einen Anzeigenplatz gibt. In diesem Fall wird der Anzeigenplatz als ungebucht bezeichnet.

Die Eingaben des Redakteurs (Publikation, Anzeigenplatz, Versandzeitpunkt, Versandmodus) und der Status der Anzeige auf Advertate Seite (Vollständigkeit und Buchungsstatus) ergeben einige zu testende Kombinationsmöglichkeiten. Die unten aufgeführte Tabelle veranschaulicht mit welchem XML-Dokument Advertate in den verschiedenen Fällen antwortet.

| Buchungsstatus | Vollständigkeit | Versandmodus | XML-Dokument                                                                                     |
|----------------|-----------------|--------------|--------------------------------------------------------------------------------------------------|
| reserviert     | unvollständig   | Vorschau     | Fehlermeldung '*Anzeigenplatz nicht gebucht*'                                                    |
| reserviert     | vollständig     | Vorschau     | Fehlermeldung '*Anzeigenplatz nicht gebucht*'                                                    |
| reserviert     | unvollständig   | Test         | Leere Anzeige                                                                                    |
| reserviert     | vollständig     | Test         | Leere Anzeige                                                                                    |
| reserviert     | unvollständig   | Echt         | Leere Anzeige                                                                                    |
| reserviert     | vollständig     | Echt         | Leere Anzeige                                                                                    |
| gebucht        | unvollständig   | Vorschau     | Fehlermeldung '*Unvollständige Anzeige*'                                                         |
| gebucht        | vollständig     | Vorschau     | Textanzeige oder Bildanzeige; je nachdem welcher Anzeigentyp auf dem Anzeigenplatz gebucht wurde |
| gebucht        | unvollständig   | Test         | Leere Anzeige                                                                                    |
| gebucht        | vollständig     | Test         | Textanzeige oder Bildanzeige; je nachdem welcher Anzeigentyp auf dem Anzeigenplatz gebucht wurde |
| gebucht        | unvollständig   | Echt         | Leere Anzeige                                                                                    |
| gebucht        | vollständig     | Echt         | Textanzeige oder Bildanzeige; je nachdem welcher Anzeigentyp auf dem Anzeigenplatz gebucht wurde |

| Anzeigenplatz   | Versandmodus | XML-Dokument                                                                                     |
|-----------------|--------------|-------------------------------------------------|
| ungebucht       | Vorschau     | Fehlermeldung '*Anzeigenplatz nicht gebucht*'   |
| ungebucht       | Test         | Leere Anzeige                                   |
| ungebucht       | Echt         | Leere Anzeige                                   |
| existiert nicht | Vorschau     | Fehlermeldung '*Anzeigenplatz nicht vorhanden*' |
| existiert nicth | Test         | Leere Anzeige                                   |
| existiert nicth | Echt         | Leere Anzeige                                   |

Ob eine vollständige/unvollständige Bild-/Textanzeige von Advertate geliefert wird, können Sie über den Anzeigenplatz Namen wählen. Die Namen entsprechen den vier Kombinationsmöglichkeiten: `Complete Image Ad`, `Incomplete Image Ad`, `Complete Text Ad`, `Incomplete Text Ad`

Den Buchungsstatus der gelieferten Anzeige wählen Sie (leider nicht ganz intuitiv) über das gewählte Versanddatum: 22-06-2015=reserviert; 29-06-2015=gebucht.
Analog wählen können Sie den Status des Anzegenplatzes wählen: 01-01-2015=existiert nicht; 15-06-2015=ungebucht;

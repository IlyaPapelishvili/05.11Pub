---
"$title": Erstellen Sie ein UI-Widget mit benutzerdefiniertem JavaScript
"$order": '101'
formats:
- Websites
tutorial: wahr
author:
- morsssss
- CrystalOnScript
description: Für Web-Erlebnisse, die einen hohen Anpassungsaufwand erfordern, hat AMP Amp-Script erstellt, eine Komponente, die die Verwendung von beliebigem JavaScript auf Ihrer AMP-Seite ermöglicht, ohne die Leistung der Seite zu beeinträchtigen.
---

In diesem Tutorial erfahren Sie, wie Sie `<amp-script>` , eine Komponente, mit der Entwickler benutzerdefiniertes JavaScript in AMP schreiben können. Sie werden dies verwenden, um ein Widget zu erstellen, das den Inhalt eines Kennworteingabefelds überprüft und nur dann übermittelt, wenn bestimmte Anforderungen erfüllt sind. AMP bietet diese Funktionalität bereits mit `<amp-form>` , aber mit `<amp-script>` können Sie eine benutzerdefinierte Erfahrung erstellen.

## Was du brauchen wirst

- Ein moderner Webbrowser
- Grundkenntnisse in HTML, CSS und JavaScript
- Entweder:
    - ein lokaler Webserver und ein Code-Editor wie [SublimeText](https://www.sublimetext.com) oder [VSCode](https://code.visualstudio.com/)
    - *oder* [CodePen](https://codepen.io/) , [Glitch](https://glitch.com/) oder ein ähnlicher Online-Spielplatz

## Hintergrund

AMP zielt darauf ab, Websites für Benutzer schneller und stabiler zu machen. Übermäßiges JavaScript kann eine Webseite verlangsamen. Manchmal müssen Sie jedoch Funktionen erstellen, die AMP-Komponenten nicht bieten. In solchen Fällen können Sie die [`<amp-script>`](../../../documentation/components/reference/amp-script.md) -Komponente verwenden, um benutzerdefiniertes JavaScript zu schreiben.

Lass uns anfangen!

# Anfangen

To get the starter code, download or clone [this github repository](https://github.com/ampproject/samples/tree/master/amp-script-tutorial). Once you've done this, `cd` into the directory you've created. You'll see two directories: `starter_code` and `finished_code`. `finished_code` contains what you'll create during this tutorial. So let's not look at that yet. Instead, `cd` into `starter_code`. This contains a webpage that implements our form using [`<amp-form>`](../../../documentation/components/reference/amp-form.md) alone, without help from `<amp-script>`.

Um diese Übung durchzuführen, müssen Sie einen Webserver auf Ihrem Computer ausführen. Wenn Sie dies bereits tun, sind Sie fertig! In diesem Fall können Sie abhängig von Ihrem Setup auf die Starter-Webseite zugreifen, indem Sie in Ihren Browser eine URL wie `http://localhost/amp-script-tutorial/starter_code/index.html` .

Alternately, you can set up a quick local server using something like [serve](https://www.npmjs.com/package/serve), a [Node.js](https://nodejs.org/)-based static content server. If you haven't installed Node.js, download it [here](https://nodejs.org/). Once Node is installed, type `npx serve` on your command line. You can then access your website here:

`http://localhost:5000/`

Sie können auch einen Online-Spielplatz wie [Glitch](https://glitch.com/) oder [CodePen nutzen](https://codepen.io/) . <a href="itch%5D(https://glitch.com/~grove-thankful-ragdoll" target="_blank">Dieser</a> enthält den gleichen Code wie das Github-Repository, und Sie können stattdessen dort beginnen, wenn Sie möchten!

Sobald Sie dies getan haben, sehen Sie unsere Starter-Webseite:

{{image ('/ static / img / docs / tutorials / custom-javascript-tutorial / Starter-form.jpg', 600, 325, layout = 'intrinsic', alt = 'Webformular mit E-Mail- und Passworteingaben', ausrichten = 'center')}}

Öffnen Sie `starter_code/index.html` in Ihrem bevorzugten Code-Editor. Schauen Sie sich den HTML-Code für dieses Formular an. Beachten Sie, dass das Kennwort `<input>` dieses Attribut enthält:

```html
on="tap:rules.show; input-debounced:rules.show"
```

Dies weist AMP an, die Regeln `<div>` wenn der Benutzer auf das Kennwort `<input>` tippt oder darauf klickt, und auch nachdem er dort ein Zeichen eingegeben hat. Wir würden es vorziehen , die verwenden `focus` Veranstaltung, die auch den Fall abdecken würde , wo die Benutzer mit der Tabulatortaste in den Eingang. Zumindest zum Zeitpunkt der Erstellung dieses Tutorials gibt AMP dieses Ereignis nicht weiter, sodass wir diese Option nicht haben. Mach dir keine Sorgen. Wir werden das mit `<amp-script>` beheben!

Das Passwort `<input>` enthält ein weiteres interessantes Attribut:

```html
pattern="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^a-z\d]).{8,}$"
```

Dieser reguläre Ausdruck kombiniert eine Reihe kleinerer regulärer Ausdrücke, von denen jeder eine unserer Validierungsregeln ausdrückt. AMP [lässt das Formular erst senden,](../../../documentation/components/reference/amp-form.md#verification) wenn der Inhalt der Eingabe übereinstimmt. Wenn der Benutzer es versucht, wird eine Fehlermeldung angezeigt, die einige Details enthält:

{{image ('/ static / img / docs / tutorials / custom-javascript-tutorial / Starter-Formularfehler.jpg', 600, 442, layout = 'intrinsic', alt = 'Webformular mit Fehlermeldung', ausrichten = 'center')}}

[tip type="note"] Da der von uns bereitgestellte Code keinen Webservice enthält, der das Senden von Formularen behandelt, ist das Senden des Formulars nicht hilfreich. Natürlich können Sie diese Funktion auch Ihrem eigenen Code hinzufügen! [/tip]

Diese Erfahrung ist akzeptabel - aber AMP kann leider nicht erklären, welche unserer Überprüfungsregeln fehlgeschlagen sind. Es kann nicht wissen, da wir die Regeln in einem einzigen regulären Ausdruck zusammenfassen mussten.

Verwenden wir jetzt `<amp-script>` , um eine benutzerfreundlichere Erfahrung zu erzielen!

# Neuaufbau mit &lt;amp-script&gt;

Um `<amp-script>` , müssen wir ein eigenes JavaScript importieren. Öffnen Sie `index.html` und fügen Sie dem `<head>` Folgendes hinzu.

```html
&lt;head&gt;
 ...
  &lt;script async custom-element="amp-script" src="https://cdn.ampproject.org/v0/amp-script-0.1.js"&gt;&lt;/script&gt;
  ...
&lt;/head&gt;

```

`<amp-script>` können wir unser eigenes JavaScript inline oder in eine externe Datei schreiben. In dieser Übung schreiben wir genug Code, um eine separate Datei zu verdienen. Erstellen Sie ein neues Verzeichnis mit dem Namen `js` und fügen Sie eine neue Datei mit dem Namen `validate.js` .

`<amp-script>` allows your JavaScript to manipulate its DOM children - the elements the component encloses. It copies those DOM children into a virtual DOM, and it gives your code access to this virtual DOM. In this exercise, we want our JavaScript to control our `<form>` and its contents. So, we'll wrap the `<form>` in an `<amp-script>` component, like this:

```html
&lt;amp-script src="js/validate.js" layout="fixed" sandbox="allow-forms" height="500" width="750"&gt;
  &lt;form method="post" action-xhr="#" target="_top" class="card"&gt;
    ...
  &lt;/form&gt;
&lt;/amp-script&gt;
```

Unser `<amp-script>` enthält das Attribut `sandbox="allow-forms"` . Das sagt AMP, dass das Skript den Inhalt des Formulars ändern kann.

Da AMP eine schnelle, visuell stabile Benutzererfahrung gewährleisten möchte, kann unser JavaScript zu keinem Zeitpunkt uneingeschränkte Änderungen am DOM vornehmen. Ihr JavaScript kann weitere Änderungen vornehmen, wenn sich die Größe der `<amp-script>` -Komponente nicht ändern kann. Es ermöglicht auch wesentlichere Änderungen nach einer Benutzerinteraktion. Details finden Sie in [der Referenzdokumentation](../../../documentation/components/reference/amp-script.md) . Für dieses Tutorial genügt es zu wissen , dass wir angegeben haben `layout` - Typ, der nicht ist `container` , und wir haben verwendeten HTML - Attribute der Komponente Größe zu sperren. Dies bedeutet, dass alle DOM-Manipulationen auf einen bestimmten Bereich der Seite beschränkt sind.

Wenn Sie die [AMP-Validator-Chrome-Erweiterung verwenden](https://chrome.google.com/webstore/detail/amp-validator/nmoffdblmcmgeicmolmhobpoocbbmknc) , wird jetzt eine Fehlermeldung angezeigt:

{{image ('/ static / img / docs / tutorials / custom-javascript-tutorial / relative-url-error.png', 600, 177, layout = 'intrinsic', alt = 'Fehler bezüglich der relativen URL', align = 'Center' ) }}

[tip type="note"] Wenn Sie diese Erweiterung nicht haben, hängen Sie `#development=1` an Ihre URL an, und AMP gibt Validierungsfehler an Ihre Konsole aus. [/tip]

Was bedeutet das? Wenn Ihr `<amp-script>` sein JavaScript aus einer externen Datei lädt, müssen Sie bei AMP eine absolute URL angeben. Wir könnten dies mithilfe von `http://localhost/js/validate.js` . AMP erfordert jedoch auch die Verwendung von [HTTPS](https://developers.google.com/web/fundamentals/security/encrypt-in-transit/why-https) . Daher wird weiterhin ein Validierungsfehler angezeigt, und das Einrichten von SSL auf unserem lokalen Webserver liegt außerhalb des Bereichs dieses Lernprogramms. Wenn Sie dies tun möchten, können Sie den Anweisungen in [diesem Beitrag](https://timonweb.com/posts/running-expressjs-server-over-https/) folgen.

Als nächstes können wir das `pattern` und seinen regulären Ausdruck aus unserem Formular entfernen: Wir werden es nicht mehr brauchen!

Wir werden auch das `on` Attribut entfernen, das derzeit verwendet wird, um AMP anzuweisen, unsere Kennwortregeln anzuzeigen. Wie oben angedeutet, werden wir stattdessen verwenden `<amp-script>` des Browsers erfassen `focus` Ereignis.

```html
pattern="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^a-z\d]).{8,}$"
on="tap:rules.show; input-debounced:rules.show"
```

Stellen wir nun sicher, dass unser `<amp-script>` funktioniert. Öffnen Sie die von Ihnen erstellte Datei `validate.js` und fügen Sie eine Debug-Nachricht hinzu:

```js
console.log("Hello, amp-script!");
```

Gehen Sie zu Ihrem Browser, öffnen Sie die Konsole und laden Sie die Seite neu. Stellen Sie sicher, dass Sie Ihre Nachricht sehen!

{{image ('/ static / img / docs / tutorials / custom-javascript-tutorial / hallo-amp-script.png', 600, 22, layout = 'intrinsic', alt = 'Hallo amp-script-Nachricht in der Konsole' , align = 'center')}}

## Wo ist mein JavaScript?

`<amp-script>` führt Ihr JavaScript in einem Web Worker aus. Web Worker können nicht direkt auf das DOM zugreifen, daher gibt `<amp-script>` dem Worker Zugriff auf eine virtuelle Kopie des DOM, die mit dem realen DOM synchronisiert bleibt. `<amp-script>` bietet Emulationen vieler gängiger DOM-APIs, von denen Sie fast alle wie gewohnt in Ihrem JavaScript verwenden können.

Wenn Sie zu irgendeinem Zeitpunkt Ihr Skript debuggen müssen, können Sie in einem Web Worker Haltepunkte in JavaScript festlegen, wie Sie es mit JavaScript tun. Sie müssen nur wissen, wo Sie es finden.

Öffnen Sie in Chrome DevTools die Registerkarte "Quellen". Unten sehen Sie eine lange hexadezimale Zeichenfolge wie die unten gezeigte. Erweitern Sie das, und erweitern Sie dann den Bereich "Keine Domain". Daraufhin wird Ihr Skript angezeigt:

{{image ('/ static / img / docs / tutorials / custom-javascript-tutorial / script-in-sources.png', 303, 277, layout = 'intrinsic', alt = 'amp-script JavaScript im DevTools Sources-Bedienfeld ', align =' center ')}}

# Hinzufügen unseres JavaScript

Jetzt, da wir wissen, dass unser `<amp-script>` funktioniert, schreiben wir etwas JavaScript!

Das erste, was wir tun möchten, ist, die DOM-Elemente, mit denen wir arbeiten, zu greifen und diese in globalen Elementen zu speichern. Unser Code verwendet die Passworteingabe, die Senden-Schaltfläche und den Bereich, in dem die Passwortregeln angezeigt werden. Fügen Sie diese drei Deklarationen zu `validate.js` :

```js
const passwordBox = document.getElementById("passwordBox");
const submitButton = document.getElementById("submitButton");
const rulesArea = document.getElementById("rules");
```

Beachten Sie, dass wir reguläre DOM-API-Methoden wie `getElementById()` . Obwohl unser Code in einem Worker ausgeführt wird und Worker keinen direkten Zugriff auf das DOM haben, stellt `<amp-script>` eine virtuelle Kopie des DOM bereit und emuliert einige der [hier](https://github.com/ampproject/worker-dom/blob/main/web_compat_table.md) aufgeführten allgemeinen APIs. Diese APIs bieten uns genügend Tools, um die meisten Anwendungsfälle abzudecken. Es ist jedoch wichtig zu beachten, dass nur eine Teilmenge der DOM-API unterstützt wird. Andernfalls wäre das in `<amp-script>` enthaltene JavaScript enorm und würde die Leistungsvorteile von AMP zunichte machen!

Wir müssen diese IDs zu zwei der Elemente hinzufügen. Öffnen Sie die `index.html` , suchen Sie das Kennwort `<input>` und die Senden- `<button>` und klicken Sie auf die IDs. Fügen Sie dem Submit- `<button>` auch ein `disabled` Attribut hinzu, damit der Benutzer nicht darauf klickt, bis er dies wünscht.

```html
&lt;input type=password
       id="passwordBox"

...

&lt;button type="submit" id="submitButton" tabindex="3" disabled&gt;Submit&lt;/button&gt;
```

Seite neu laden. Sie können überprüfen, ob diese Globals korrekt festgelegt wurden, indem Sie in der Konsole einchecken, genau wie Sie es mit Nicht-Worker-JavaScript tun könnten:

{{image ('/ static / img / docs / tutorials / custom-javascript-tutorial / global-set.png', 563, 38, layout = 'intrinsic', alt = 'Konsolennachricht, die zeigt, dass submitButton gesetzt ist', align = 'Center' ) }}

Wir werden auch jedem `<li>` in `<div id="rules">` hinzufügen. Jede dieser Regeln enthält eine individuelle Regel, deren Farbe wir steuern möchten. Und wir werden jede Instanz von `class="invalid"` entfernen. Unser neues JavaScript wird das hinzufügen, wenn es benötigt wird!

```html
&lt;ul&gt;
  &lt;li id="lower"&gt;Lowercase letter&lt;/li&gt;
  &lt;li id="upper"&gt;Capital letter&lt;/li&gt;
  &lt;li id="digit"&gt;Digit&lt;/li&gt;
  &lt;li id="special"&gt;Special character (@$!%*?&amp;)&lt;/li&gt;
  &lt;li id="eight"&gt;At least 8 characters long&lt;/li&gt;
&lt;/ul&gt;
```

## Implementierung unserer Passwortprüfungen in JavaScript

Als nächstes entpacken wir die regulären Ausdrücke aus unserem `pattern` . Jeder reguläre Ausdruck repräsentiert eine unserer Regeln. Fügen wir am Ende von `validate.js` eine Objektzuordnung hinzu, die jede Regel dem von ihr überprüften Kriterium zuordnet.

```js
const checkRegexes = {
  lower: /[a-z]/,
  upper: /[A-Z]/,
  digit: /\d/,
  special: /[^a-zA-Z\d]/i,
  eight: /.{8}/
};
```

With those globals set, we're ready to write the logic that checks the password and adjusts the UI accordingly. We'll put our logic inside a function called `initCheckPassword` that takes a single argument - the DOM element of the password `<input>`. This approach conveniently stashes the DOM element in a closure.

```js
function initCheckPassword(element) {

}
```

Next, let's populate `initCheckPassword` with the functions and event listener assignments we'll need. First of all, add a small function that turns an individual rule `<li>` green if the rule passes - and another that turns it red when it fails.

```js
function initCheckPassword(el) {
  const checkPass = (el) =&gt; {
    el.classList.remove("invalid");
    el.classList.add("valid");
  };

  const checkFail = (el) =&gt; {
    el.classList.remove("valid");
    el.classList.add("invalid");
  };
};
```

Lassen Sie uns diese `valid` und `invalid` Klassen tatsächlich grün oder rot werden lassen. Gehen Sie zurück zu `index.html` und fügen Sie diese beiden Regeln dem Tag `<style amp-custom>` :

```css
li.valid {
  color: #2d7b1f;
}

li.invalid {
  color:#c11136;
}
```

Now we're ready to add the logic that checks the contents of the password `<input>` against our rules. Add a new function called `checkPassword()` to `initCheckPassword()`, right before the closing brace:

```js
const checkPassword = () =&gt; {
  const password = element.value;
  let failed = false;

  for (const check in checkRegexes) {
    let li = document.getElementById(check);

    if (password.match(checkRegexes[check])) {
      checkPass(li);
    } else {
      checkFail(li);
      failed = true;
    }
  }

  if (!failed) {
    submitButton.removeAttribute("disabled");
  }
};
```

Diese Funktion führt Folgendes aus:

1. Ruft den Inhalt des Passworts `<input>` .
2. Erstellt ein Flag namens " `failed` , das auf " `false` initialisiert wurde.
3. Durchläuft jeden unserer regulären Ausdrücke und testet jeden anhand des Passworts:
    - Wenn das Kennwort einen Test nicht besteht, rufen Sie `checkFail()` auf, um die entsprechende Regel rot zu machen. Außerdem konnte set `failed` auf `true` .
    - Wenn das Passwort einen Test besteht, rufen Sie `checkPass()` auf, um die entsprechende Regel grün zu machen.
4. Wenn keine Regel fehlgeschlagen ist, ist das Passwort gültig und wir aktivieren die Schaltfläche Senden.

Jetzt brauchen wir nur noch ein paar Event-Listener. Denken Sie daran , wie wir waren nicht in der Lage zu verwenden `focus` Ereignis in AMP? In `<amp-script>` können wir. Jedes Mal , wenn das Passwort `<input>` die empfängt `focus` - Ereignis, werden wir die Regeln anzuzeigen. Und wenn der Benutzer eine Taste in dieser Eingabe drückt, rufen wir `checkPassword()` .

Fügen Sie diese beiden Ereignis-Listener direkt vor der schließenden Klammer am Ende von `initCheckPassword()` :

```js
element.addEventListener("focus", () =&gt; rulesArea.removeAttribute("hidden"));
element.addEventListener("keyup", checkPassword);
```

Finally, at the very end of `validate.js`, add a line that initializes `initCheckPassword` with the password `<input>` DOM element:

```js
initCheckPassword(passwordBox);
```

Unsere Logik ist jetzt vollständig! Wenn das Passwort allen unseren Kriterien entspricht, sind alle Regeln grün und unser Senden-Button wird aktiviert. Sie sollten jetzt in der Lage sein, eine Interaktion wie diese zu haben:

<figure class="alignment-wrapper margin-">
  <amp-video width="762" height="564" layout="responsive" autoplay loop noaudio>
    <source src="/static/img/docs/tutorials/custom-javascript-tutorial/finished-project.mp4" type="video/mp4">
    <source src="/static/img/docs/tutorials/custom-javascript-tutorial/finished-project.webm" type="video/webm">
  </amp-video>
</figure>

Wenn Sie nicht weiterkommen, können Sie immer auf dem Arbeits Code verweisen im `finished_code` Verzeichnis.

# Herzliche Glückwünsche!

Sie haben gelernt, wie Sie mit `<amp-script>` Ihr eigenes JavaScript in AMP schreiben. Es ist Ihnen gelungen, die `<amp-form>` -Komponente mit Ihrer eigenen benutzerdefinierten Logik und UI-Funktionen zu erweitern! Fühlen Sie sich frei, Ihrer neuen Seite weitere Funktionen hinzuzufügen! Weitere Informationen zu `<amp-script>` finden Sie in [der Referenzdokumentation](../../../documentation/components/reference/amp-script.md) .

# Migros Hör-Box

Die Migros Hör-Box wird für das Abspielen von Hörspielen von Storymania oder den Weihnachts-Hörspielen verwendet.
Durch platzieren einer bestimmten Figur auf der Hör-Box wird ein passendes Hörspiel gespielt.
Leider sind ein paar Tricks nötig um seine eigenen Audiofiles auf der Box abspielen zu können.

## Funktion der Hörbox
Die Figuren besitzen in ihrem Sockel einen NFC Chip. Dadurch kann die Hör-Box eine zur Figur passende Audiodatei abspielen.
Für jede Figur muss sich somit eine Audiodatei (M01.smp, M02.smp, ...) auf der Box befinden.
Leider sind die Audiodateien leicht modifizierte MP3 Dateien.
Es handelt sich hier um eine sipmle XOR Codierung, die immer wieder als billigen Ersatz für Verschlüsselung verwendet wird.
Da XOR umkehrbar ist kann man mit der selben operation aus den .smp Dateien einfach .mp3 Dateien herstellen, jedoch auch aus seinen eigenen .mp3 Dateien .smp Dateien herstellen. Somit kann man mit wenig Aufwand die Lieblings-CD der Kinder auf die Box bringen.


## Vorbereitung
Als erstes benötigt man die Lieblings-CD seiner Kinder als MP3.
Da normalerweise eine CD in mehrere Dateien unterteilt ist, muss man die einzelnen .mp3 Dateien zuerst zusammen mergen:

    sudo apt-get install mp3wrap
    mp3wrap merged.mp3 *.mp3

Da die Box nur über einen Lautsprecher verfügt, kann die Datei ruhig als mono codiert werden. Ich verwende dafür den Sound Converter:

    sudo apt-get install soundconverter

Unter den Preferences kann "Force mono output" ausgewählt werden. Die Hör-Box unterstützt keine variable Bitrate (VBR).


## Konvertierung
Für die Konvertierung benötigt man ein XOR Tool:

    git clone https://github.com/hellman/xortool.git
    cd xortool
    sudo python setup.py install

Die vorbereitete .mp3 Datei muss nun lediglich mit dem ASCII-Wert "f" codiert werden:

    xortool/xortool-xor -f merged_MP3WRAP.mp3 -s "f" > M01.smp

Somit kann jede Figur (M01 bis M??.smp") mit einer eingenen Audiodatei belegt werden.

## Forensik
Wie kommt man nun darauf, wie man die .smp Dateien auf der Box codieren muss?
Hier eine etwas technische Herleitung.

Als erstes schauen wir mal, was das Unix-Helferchen ´file´ zu den Dateien auf der Box meint:

    % file *.smp
    M01.smp:  data
    M02.smp:  data
    ...

Leider werden die Files nicht erkannt.
Schaut man sich nun die Dateien etwas ganauer an, merkt man, dass sie alle mit der gleichen Header-Sequenz beginnen:

    % for f in M??.smp; do hexdump -C $f | head -n 1; done
    00000000  99 9c b6 66 11 02 66 66  66 66 66 66 66 66 66 66  |...f..ffffffffff|
    00000000  99 9c b6 66 11 02 66 66  66 66 66 66 66 66 66 66  |...f..ffffffffff|
    00000000  99 9c b6 66 11 02 66 66  66 66 66 66 66 66 66 66  |...f..ffffffffff|
    00000000  99 9c b6 66 11 02 66 66  66 66 66 66 66 66 66 66  |...f..ffffffffff|
    00000000  99 9c b6 66 11 02 66 66  66 66 66 66 66 66 66 66  |...f..ffffffffff|
    00000000  99 9c b6 66 11 02 66 66  66 66 66 66 66 66 66 66  |...f..ffffffffff|
    00000000  99 9c b6 66 11 02 66 66  66 66 66 66 66 66 66 66  |...f..ffffffffff|
    00000000  99 9c b6 66 11 02 66 66  66 66 66 66 66 66 66 66  |...f..ffffffffff|
    00000000  99 9c b6 66 11 02 66 66  66 66 66 66 66 66 66 66  |...f..ffffffffff|
    00000000  99 9c b6 66 11 02 66 66  66 66 66 66 66 66 66 66  |...f..ffffffffff|
    00000000  99 9c b6 66 11 02 66 66  66 66 66 66 66 66 66 66  |...f..ffffffffff|

Ein erster Check auf  [Wikipedia](https://en.wikipedia.org/wiki/List_of_file_signatures) und eine Google-Suche bringen jedoch noch keine Resultate für ein Dateiformat das mit der Bytefolge "99 9c" beginnt.

Als zweites sticht einem natürlich die vielen 66 ins Auge. Dies ist eher ungewöhnlich, da Binary-Dateien nach einer definierten Header-Sequenz oft ein paar 00 enthalten, bis an einer bestimmten Stelle der eignentliche Payload beginnt. Dies könnte also darauf hindeuten dass die Dateien mit einer Byte-weisen Transformation codiert wurden. Das naheliegenste ist natürlich XOR. Um von einem Byte auf 00 zu kommen muss es mit sich selber XOR codieren, in unserem Fall also 66 was dem ASCII-Wert "f" entspricht.

    for f in M??.smp; do xortool/xortool-xor -f $f -s "f" > $f.xor; done

    file *.smp.xor
    M01.smp.xor: MPEG ADTS, layer III, v1, 256 kbps, 44.1 kHz, Stereo
    M02.smp.xor: MPEG ADTS, layer III, v1, 256 kbps, 44.1 kHz, Stereo
    ...

Voilà! Wir haben MPEG layer III, also MP3 ...

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

    sudo apt-get install 
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


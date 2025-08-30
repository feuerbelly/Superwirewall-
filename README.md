# Superwirewall-
"Eine visuelle 3D-Simulation einer selbstheilenden Firewall, basierend auf dem Konzept der Digitalen DNA."

from ursina import *
import random, math, time
import hashlib

# ----------------------------------------------------------------------------------
# Simulation einer vernetzten Sicherheitsarchitektur im 3D-Raum
# Das System nutzt "Knoten" mit unterschiedlichen Funktionen, um sich gegen Angriffe zu verteidigen.
# Metapher: "Digitale DNA" als Basis für den gesunden Zustand.
# ----------------------------------------------------------------------------------

# --- Digitale DNA ---
original_dna = {f"Knoten_{x}_{y}_{z}": 5
                for x in range(-3,4,2)
                for y in range(0,5,2)
                for z in range(-3,4,2)}
original_hash = hashlib.sha256(str(original_dna).encode()).hexdigest()

# --- Muster für jeden Knoten ---
knoten_muster = {}
knoten_muster["Knoten_-3_0_-3_A"] = "Honeypot"
knoten_muster["Knoten_-3_0_-3_B"] = "Datenanalyst"
knoten_muster["Knoten_3_4_3_A"] = "Virenscanner"
knoten_muster["Knoten_3_4_3_B"] = "Datenanalyst"

for name in [f"Knoten_{x}_{y}_{z}_A" for x in range(-3,4,2) for y in range(0,5,2) for z in range(-3,4,2)]:
    if name not in knoten_muster: knoten_muster[name] = "emotion"
for name in [f"Knoten_{x}_{y}_{z}_B" for x in range(-3,4,2) for y in range(0,5,2) for z in range(-3,4,2)]:
    if name not in knoten_muster: knoten_muster[name] = "emotion"

app = Ursina()

# --- Farben ---
HELLBLAU = color.rgb(137,207,240)
ENTKOPPELT = color.red
GEDICHT_COLOR = color.yellow
LINIE_STABIL = color.white
LINIE_FLACKER = color.rgb(200,200,255)

# --- Erweiterte Gedichte ---
zeilen_warnung = [
    "Klock klock, Gefahr naht!", "Halt! Etwas stimmt hier nicht…", "Vorsicht, Fremdes erkannt!",
    "Ein Schatten schleicht durch die Leitungen", "Alarm! Etwas flackert im Netz",
    "Virenpfad entdeckt, bitte Abstand halten", "Fehler tanzt auf unsichtbaren Pfaden",
    "Stille vor dem Sturm, achte auf jede Regung", "Warnung! Ein Knoten bricht das Band",
    "Dunkle Funken fliegen durch die Adern", "Gefahr kichert hinter der Datenwand",
    "Ein unerwarteter Puls erschüttert die Struktur", "Vorsicht, unsichtbare Hand greift",
    "Flackernde Lichter zeigen Unruhe an", "Der Fluss der Sicherheit stockt kurz",
    "Ein Virus versucht, sich einzuschleichen", "Knoten auf Abwegen – Alarm aktiviert",
    "Störungen summen wie leise Geister", "Signalverlust droht, sei achtsam", "Netzwerk atmet unruhig"
]

zeilen_heilung = [
    "Sanftes Licht heilt die Wunde", "Alles findet zurück zur Ruhe", "Balance kehrt zurück",
    "Wellen der Harmonie umspülen die Knoten", "Die Ordnung kehrt zurück, still und sanft",
    "Verborgene Energie stärkt das System", "Jeder Knoten atmet neue Kraft ein",
    "Licht durchströmt jede Leitung", "Sanfte Schwingungen glätten die Pfade",
    "Knoten verbinden sich in Frieden", "Stille heilt die Wunden im Netzwerk",
    "Harmonie fließt wie Wasser durch Adern", "Funkeln der Stabilität kehren zurück",
    "Daten tanzen wieder synchron", "Schutzschild erneuert sich leise", "Alles pulsiert im Gleichklang",
    "Wiederherstellung beginnt, sanft und leise", "Jede Lücke wird gefüllt mit Licht", "Ruhe zieht durch die Räume",
    "Vertrauen kehrt in die Leitungen zurück"
]

zeilen_freude = [
    "Schatten tanzen, doch Licht lacht", "Freude fließt durch jedes Band", "Alles pulsiert in Harmonie",
    "Lichtfunken tanzen durch die Adern", "Knoten lachen leise im Takt der Daten",
    "Glückseligkeit breitet sich aus", "Strahlen der Freude leuchten auf jedem Knoten",
    "Ein Regen aus Licht umspielt die Leitungen", "Knoten summen vor Zufriedenheit",
    "Jeder Impuls singt ein kleines Lied", "Daten wirbeln in beschwingtem Tanz",
    "Freundliche Energie durchflutet das System", "Lichter blitzen wie funkelnde Sterne",
    "Glück tanzt durch die Räume", "Alles vibriert in strahlender Verbindung",
    "Sanfte Freude umhüllt jeden Knoten", "Leuchtende Funken verbreiten Leichtigkeit",
    "Netzwerk pulsiert in rhythmischer Freude", "Licht und Farbe fließen wie Musik", "Jeder Knoten summt ein Dankeslied"
]

def kombiniere_gedicht(stimmung="warnung"):
    if stimmung=="warnung":
        return random.choice(zeilen_warnung) + " " + random.choice(zeilen_warnung)
    elif stimmung=="heilung":
        return random.choice(zeilen_heilung) + " " + random.choice(zeilen_heilung)
    else:
        return random.choice(zeilen_freude) + " " + random.choice(zeilen_freude)

# --- Datenpakete ---
def generate_data_packet():
    typ = random.choice(["emotion", "virus", "daten"])
    wert = random.randint(1,10)
    return {"typ":typ, "wert":wert}

# --- Knoten ---
class Knoten(Entity):
    def __init__(self, name, position, basiswert, partner=None):
        super().__init__(model='sphere', scale=0.5, position=position)
        self.name = name
        self.basiswert = basiswert
        self.partner = partner
        self.entkoppelt = False
        self.text_entity = None
        self.timer = 0
        self.typ = knoten_muster.get(self.name,"emotion")
        self.virus_welle = None

        if self.typ=="Virenscanner": self.color=color.lime
        elif self.typ=="Datenanalyst": self.color=color.orange
        elif self.typ=="Honeypot": self.color=color.gold
        else: self.color=HELLBLAU

    def pruefe_wert(self,paket):
        if self.entkoppelt:
            self.timer += time.dt
            self.color = ENTKOPPELT.tint(0.5)
            self.scale = 0.5 + 0.1*math.sin(time.time()*5)
            if self.timer>5:
                self.entkoppelt=False
                self.color = color.lime if self.typ=="Virenscanner" else (color.orange if self.typ=="Datenanalyst" else (color.gold if self.typ=="Honeypot" else HELLBLAU))
                self.pop_gedicht("heilung")
                self.timer=0
            return
        self.individueller_algorithmus(paket)

    def individueller_algorithmus(self,paket):
        wert = paket["wert"]
        typ = paket["typ"]

        if self.typ=="Honeypot":
            if typ=="virus":
                print(f"*** Honeypot {self.name} hat Virus gefangen! ***")
                self.pop_gedicht("warnung")
                self.virus_welle = Entity(model='sphere', scale=0.1, color=color.red, position=self.position)
                self.virus_welle.animate_scale(5,duration=1)
                invoke(destroy,self.virus_welle,delay=1)
                return
            else:
                self.color=color.gold
                return

        if self.typ=="Virenscanner":
            if typ=="virus":
                self.entkoppelt=True
                self.pop_gedicht("warnung")
                if self.partner: self.partner.analyse_fehler(self)
                return
            else:
                self.color=color.lime.tint(0.5)
                return

        if self.typ=="Datenanalyst":
            if typ=="daten":
                self.basiswert=(self.basiswert+wert)/2
                self.color=color.orange.tint(0.5)
                return
            else:
                self.entkoppelt=True
                self.pop_gedicht("warnung")
                if self.partner: self.partner.analyse_fehler(self)
                return

        else:
            partner_wert=self.partner.basiswert if self.partner else self.basiswert
            if wert==self.basiswert and wert==partner_wert:
                self.scale=0.5+0.05*math.sin(time.time()*3+hash(self.name)%10)
                blend=(math.sin(time.time()*2+hash(self.name)%10)+1)/2
                self.color=lerp(HELLBLAU,color.white,blend)
            else:
                self.entkoppelt=True
                self.color=ENTKOPPELT
                self.pop_gedicht("warnung")
                if self.partner: self.partner.analyse_fehler(self)

    def analyse_fehler(self,ausgefallen):
        original_name=ausgefallen.name.replace("_A","").replace("_B","")
        ausgefallen.basiswert=original_dna.get(original_name,5)
        print(f"Wiederhergestellt: {ausgefallen.name}")

    def pop_gedicht(self,stimmung="warnung"):
        text = kombiniere_gedicht(stimmung)
        self.text_entity = Text(text, position=self.position+Vec3(0,0.7,0),
                                origin=(0,0), color=GEDICHT_COLOR, scale=1)
        self.text_entity.animate_y(self.text_entity.y + 0.5, duration=1.5, curve=curve.out_expo)
        self.text_entity.animate_color(color.white, duration=1.5, curve=curve.sin)
        for _ in range(5):
            p = Entity(model='sphere', color=color.yellow, scale=0.05,
                       position=self.position+Vec3(random.uniform(-0.2,0.2),0,random.uniform(-0.2,0.2)))
            p.animate_y(p.y + random.uniform(0.5,1.0), duration=1, curve=curve.linear)
            invoke(destroy, p, delay=1.5)
        invoke(destroy, self.text_entity, delay=3)

# ----------------------------------------------------------------------------------
# Simulation starten
# ----------------------------------------------------------------------------------

# Erstellen der Knoten im 3D-Gitter.
knoten_3d = []
for x in range(-3,4,2):
    for y in range(0,5,2):
        for z in range(-3,4,2):
            name1 = f"Knoten_{x}_{y}_{z}_A"
            name2 = f"Knoten_{x}_{y}_{z}_B"
            basiswert = original_dna.get(f"Knoten_{x}_{y}_{z}",5)
            kn1 = Knoten(name1, Vec3(x,y,z), basiswert)
            kn2 = Knoten(name2, Vec3(x+0.2,y+0.2,z+0.2), basiswert)
            kn1.partner = kn2
            kn2.partner = kn1
            knoten_3d.extend([kn1, kn2])

# Linien
linien = []
for k1 in knoten_3d:
    for k2 in knoten_3d:
        if k1.position != k2.position and distance(k1.position, k2.position) < 3.0:
            linie = Entity(model=Mesh(vertices=[k1.position, k2.position], mode='line'), color=LINIE_STABIL)
            linien.append(linie)

# Hauptloop
def update():
    for k in knoten_3d:
        k.pruefe_wert(generate_data_packet())
    
app.run()

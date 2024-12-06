Lern-Periode 2
25.10 bis 20.12

In dieser Lernperiode möchte ich etwas ganz Neues und Einzigartiges entwickeln. Vielleicht wird es ein spannendes 
Unity Spiel mit besonderen Funktionen und herausfordernden Mechaniken, die die Spieler überraschen. Oder ich schreibe eine komplett neue Anwendung
die so bisher noch nicht existiert und den Alltag erleichtert. Beide Ideen sind anspruchsvoll und werden mir helfen, meine Programmierfähigkeiten weiterzuentwickeln.
Ich werde meine Module genau analysieren, um zu sehen, wo meine Stärken liegen und welche Schwächen ich noch verbessern kann.


25.10.2024

Meine Noten sind durchschnittlich, im Allgemeinen gibt es keinen Teil, in dem ich große Schwierigkeiten habe. Ich muss mich verbessern und weiter verbessern, um an dieser Schule zu bleiben. Ich möchte so sehr an dieser Schule bleiben, dass ich so viel arbeite, wie ich kann. 

Heute habe ich im Internet nach Ideen gesucht, ich habe nach ein paar Anwendungen gesucht, ich konnte sie nicht finden, und schließlich habe ich beschlossen, ein einfaches Spiel zu machen, das ich in einem Tag als Übung für diese Woche fertigstellen konnte. Ich habe die Bild-, Sound- und Animationspakete heruntergeladen und Flapy Bird mit Unty codiert, wenn es das Limit nicht überschreitet, sind das fertige Spiel und die Codes unten. nächste Woche kann ich alle Bilder und Figuren separat mit meiner eigenen Zeichnung machen oder ich kann ein anderes Projekt beginnen.


<img width="1728" alt="Screenshot 2024-11-01 at 11 45 28" src="https://github.com/user-attachments/assets/bb9ba1bb-0b9a-45ba-9235-4e7db2e4a1f6">
<img width="1728" alt="Screenshot 2024-11-01 at 11 45 35" src="https://github.com/user-attachments/assets/c47f4734-2041-49e1-90e2-c4e5a8d3ce7e">



[Uploadiusing UnityEngine;
using UnityEngine.UI;

[DefaultExecutionOrder(-1)]
public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }
    [SerializeField] private Player player;
    [SerializeField] private Spawner spawner;
    [SerializeField] private Text scoreText;
    [SerializeField] private GameObject playButton;
    [SerializeField] private GameObject gameOver;

    public int score { get; private set; } = 0;

    private void Awake()
    {
        if (Instance != null) {
            DestroyImmediate(gameObject);
        } else {
            Instance = this;
        }
    }

    private void OnDestroy()
    {
        if (Instance == this) {
            Instance = null;
        }
    }

    private void Start()
    {
        Pause();
    }

    public void Pause()
    {
        Time.timeScale = 0f;
        player.enabled = false;
    }

    public void Play()
    {
        score = 0;
        scoreText.text = score.ToString();

        playButton.SetActive(false);
        gameOver.SetActive(false);

        Time.timeScale = 1f;
        player.enabled = true;

        Pipes[] pipes = FindObjectsOfType<Pipes>();

        for (int i = 0; i < pipes.Length; i++) {
            Destroy(pipes[i].gameObject);
        }
    }

    public void GameOver()
    {
        playButton.SetActive(true);
        gameOver.SetActive(true);

        Pause();
    }

    public void IncreaseScore()
    {
        score++;
        scoreText.text = score.ToString();
    }

}

using UnityEngine;

public class Parallax : MonoBehaviour
{
    public float animationSpeed = 1f;
    private MeshRenderer meshRenderer;

    private void Awake()
    {
        meshRenderer = GetComponent<MeshRenderer>();
    }

    private void Update()
    {
        meshRenderer.material.mainTextureOffset += new Vector2(animationSpeed * Time.deltaTime, 0);
    }

}

using UnityEngine;

public class Pipes : MonoBehaviour
{
    public Transform top;
    public Transform bottom;
    public float speed = 5f;
    public float gap = 3f;

    private float leftEdge;

    private void Start()
    {
        leftEdge = Camera.main.ScreenToWorldPoint(Vector3.zero).x - 1f;
        top.position += Vector3.up * gap / 2;
        bottom.position += Vector3.down * gap / 2;
    }

    private void Update()
    {
        transform.position += speed * Time.deltaTime * Vector3.left;

        if (transform.position.x < leftEdge) {
            Destroy(gameObject);
        }
    }

}

using UnityEngine;

public class Player : MonoBehaviour
{
    public Sprite[] sprites;
    public float strength = 5f;
    public float gravity = -9.81f;
    public float tilt = 5f;

    private SpriteRenderer spriteRenderer;
    private Vector3 direction;
    private int spriteIndex;

    private void Awake()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
    }

    private void Start()
    {
        InvokeRepeating(nameof(AnimateSprite), 0.15f, 0.15f);
    }

    private void OnEnable()
    {
        Vector3 position = transform.position;
        position.y = 0f;
        transform.position = position;
        direction = Vector3.zero;
    }

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space) || Input.GetMouseButtonDown(0)) {
            direction = Vector3.up * strength;
        }

        // Apply gravity and update the position
        direction.y += gravity * Time.deltaTime;
        transform.position += direction * Time.deltaTime;

        // Tilt the bird based on the direction
        Vector3 rotation = transform.eulerAngles;
        rotation.z = direction.y * tilt;
        transform.eulerAngles = rotation;
    }

    private void AnimateSprite()
    {
        spriteIndex++;

        if (spriteIndex >= sprites.Length) {
            spriteIndex = 0;
        }

        if (spriteIndex < sprites.Length && spriteIndex >= 0) {
            spriteRenderer.sprite = sprites[spriteIndex];
        }
    }

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.gameObject.CompareTag("Obstacle")) {
            GameManager.Instance.GameOver();
        } else if (other.gameObject.CompareTag("Scoring")) {
            GameManager.Instance.IncreaseScore();
        }
    }

}

using UnityEngine;

public class Spawner : MonoBehaviour
{
    public Pipes prefab;
    public float spawnRate = 1f;
    public float minHeight = -1f;
    public float maxHeight = 2f;
    public float verticalGap = 3f;

    private void OnEnable()
    {
        InvokeRepeating(nameof(Spawn), spawnRate, spawnRate);
    }

    private void OnDisable()
    {
        CancelInvoke(nameof(Spawn));
    }

    private void Spawn()
    {
        Pipes pipes = Instantiate(prefab, transform.position, Quaternion.identity);
        pipes.transform.position += Vector3.up * Random.Range(minHeight, maxHeight);
        pipes.gap = verticalGap;
    }

}
ng GameManager.cs…]()






Da ich bis heute verschiedene Anwendungen und Spiele gemacht habe, sind sie jetzt langweilig und kurz. Ich mache Pläne, um ein Spiel oder eine Anwendung zu machen, die länger dauern und fast einzigartig sein wird.













8.11.2024

Technische Probleme beheben: Arbeite daran, das Problem mit Rider zu lösen, damit du das Multiplattform-Projekt starten kannst.
Marktanalyse und Ideenfindung: Recherchiere intensiv über Anwendungen, die Marktpreisvorhersagen für Aktien und Kryptowährungen machen, und notiere dir relevante Funktionen und Erfolgsfaktoren.
Alte Projekte durchsehen: Analysiere deine früheren Projekte, um herauszufinden, welche Teile du für die neue Anwendung wiederverwenden oder weiterentwickeln könntest.


Nächste Woche Planung:

Projektentwicklung starten: Sobald das technische Problem gelöst ist, beginne mit der Programmierung deiner Anwendung.
Alternative Entwicklungsumgebung prüfen: Falls das Problem mit Rider nicht gelöst werden kann, wechsle zu einer anderen Entwicklungsumgebung, die Multiplattform-Unterstützung bietet.
Entwicklungsplan erstellen: Skizziere einen detaillierten Plan für die Funktionen und Struktur deiner Anwendung. Lege Ziele und Meilensteine für die ersten Entwicklungsschritte fest, um fokussiert und effizient voranzukommen.
Mit diesem Plan kannst du in dieser Woche die Grundlagen schaffen, um nächste Woche erfolgreich mit der Programmierung zu beginnen.


15.11.2024

Für mein neues 3D Projekt habe ich viele Vorbereitungen getroffen. Zuerst habe ich recherchiert, wie man 3D Objekte erstellt, programmiert und gestaltet. Dazu habe ich verschiedene Quellen im Internet gelesen und Videos angeschaut. Besonders habe ich mich mit Tools wie Unity und Blender beschäftigt, um sie besser zu verstehen und anwenden zu können. Zusätzlich habe ich meine grundlegenden C#-Kenntnisse verbessert, indem ich einige Beispielprojekte analysiert habe.

Um noch mehr zu lernen, habe ich auch meine alten Projekte angeschaut und den Code überprüft. Dabei habe ich Fehler entdeckt, die ich damals gemacht habe, und darüber nachgedacht, wie ich daraus lernen kann. Ich habe erkannt, wie wichtig es ist, den Code gut zu strukturieren und mit klaren Kommentaren zu versehen.

Diese Vorbereitungen haben mich motiviert, und ich fühle mich jetzt sicherer, an meinem neuen Projekt zu arbeiten. Mein Plan ist es, Schritt für Schritt vorzugehen und mein Projekt so kreativ und solide wie möglich zu gestalten. Alles, was ich lerne, schreibe ich auf, damit ich es später nachschlagen kann.


22.11.2024


Ich habe die unfertige Hypothekenrechner-Anwendung überprüft und verbessert. 
Neue Codes berechnen die monatlichen Raten genauer, und Bilder sowie Diagramme machen die Ergebnisse übersichtlicher. 
Zudem erleichtern Hilfetexte die Nutzung. Die Anwendung ist jetzt funktionaler, aber noch nicht fertig.
<img width="1728" alt="Screenshot 2024-11-22 at 11 21 17" src="https://github.com/user-attachments/assets/eccf800c-048e-44b3-8da5-62dee00aacc0">


Nächste Woche werde ich mit dem Projekt weitermachen und es beenden. Sobald ich es fertiggestellt habe,
werde ich ein neues Projekt suchen und damit anfangen. Ich freue mich darauf, 
neue Herausforderungen anzugehen und mich weiterzuentwickeln.


29.11.2024

Ich habe meine Arbeit an der Hypothekenrechner-Anwendung fortgesetzt und mehrere Verbesserungen vorgenommen:

Individuelles Design für Buttons:
Um keine vorgefertigten Bilder zu verwenden, habe ich meine eigenen Buttons gestaltet. Dadurch wirkt die Benutzeroberfläche persönlicher und ansprechender.
Fehlerbehebung:
Ich habe Probleme in der Anwendung identifiziert und behoben, wodurch die Funktionen jetzt einwandfrei laufen.
Neue Funktion „Wunschhypothek“ hinzugefügt:
Ich habe eine zusätzliche Funktion implementiert, mit der Benutzer ihre gewünschte Hypothek individuell berechnen können.
Diese Anpassungen und Erweiterungen verbessern sowohl die Funktionalität als auch das Design meiner Anwendung und bieten den Nutzern eine bessere Erfahrung.

Nächste Woche werde ich die Anwendung weiterentwickeln und neue Funktionen hinzufügen, um die Benutzererfahrung noch weiter zu verbessern.


<img width="1720" alt="Screenshot 2024-11-29 at 12 00 03" src="https://github.com/user-attachments/assets/70c8be63-6be9-4405-8225-a59fa92bff82">
<img width="1699" alt="Screenshot 2024-11-29 at 11 59 47" src="https://github.com/user-attachments/assets/c2061d26-8f01-4406-83a3-a858aff37386">



06.12.2024 



Während ich viele Spielideen im Kopf hatte, suchte ich nach einem Projekt, das relativ schnell abgeschlossen werden kann.
Da erinnerte ich mich an eine Idee, die mir vor einigen Jahren kam, als das Spiel 2048 aufkam. Mein Plan war,
dieses Spielprinzip mit einigen Änderungen zu verbessern.

Das Spiel wird anstelle eines quadratischen Gitters auf einem dreieckigen Gitter gespielt. Die Zahlen werden nicht als 
Potenzen von 2 dargestellt, sondern als Potenzen von 3. Das Ziel des Spiels ist es, die Zahl 177147 zu erreichen, die 3^11 entspricht.
Zusätzlich werden die Steine in drei Richtungen bewegbar sein, was das Spiel spannender und strategischer macht.
Nachdem ich viel im Internet recherchiert hatte, wurde mir klar, dass es noch nicht genau so war, wie ich dachte. 
Diese Woche habe ich mit den Vorbereitungen für das Programmieren begonnen. Ich habe erste Entwürfe gemacht, ähnliche Spiele untersucht
und die Spielmechanik verstanden.


<img width="1728" alt="Screenshot 2024-12-06 at 11 51 00" src="https://github.com/user-attachments/assets/639463f6-046d-402f-80b5-d22e417451bf">
<img width="295" alt="Screenshot 2024-12-06 at 11 48 41" src="https://github.com/user-attachments/assets/381d970b-10fd-48ef-9d17-e87227e26685">
<img width="656" alt="Screenshot 2024-12-06 at 11 34 30" src="https://github.com/user-attachments/assets/9c4e875c-5bd3-4a94-968e-461800ce96cc">
<img width="675" alt="Screenshot 2024-12-06 at 11 52 06" src="https://github.com/user-attachments/assets/62224464-f632-45cb-b111-4274a773db22">



Nachste Woche: In der nächsten Woche werde ich mit dem Coden anfangen und plane, das Spiel innerhalb von zwei Wochen
fertigzustellen. Ich freue mich darauf, diese Idee in die Realität umzusetzen!




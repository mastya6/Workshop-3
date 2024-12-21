# АНАЛИЗ ДАННЫХ В РАЗРАБОТКЕ ИГР [in GameDev]
Отчет по лабораторной работе #3 выполнила:
- Коробейникова Анастасия Денисовна
- НМТ-231805

Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Предложите вариант изменения найденных переменных для 10 уровней в игре. Визуализируйте изменение уровня сложности в таблице.
- Задание 2.
- Создайте 10 сцен на Unity с изменяющимся уровнем сложности.
- Задание 3.
- Решение в 80+ баллов должно визуализировать данные из google-таблицы, и с помощью Python передавать в проект Unity.
  В Python данные также должны быть визуализированы.
- Выводы.

## Цель работы
Разработать оптимальный баланс изменения сложности и изменить структуру проекта, добавить усложнение.




## Задание 1
### Изменение переменных

#### Переменные которые я взяла:

-Speed - определяет скорость движния дракона

-TimeBetweenEggDrops - определяет в какие промежутки дракон бросает яйцо

-ChanceDirection - определяет с какой вероятностью дракон сменит направление

![garph1](https://github.com/mastya6/Workshop-3/blob/main/graph1.png)
![graph2](https://github.com/mastya6/Workshop-3/blob/main/graph2.png)


## Задание 2
###  Сцены Unity

Скопировала 10 раз стандратную сцену и меняла вручную сложность
![level](https://github.com/mastya6/Workshop-3/blob/main/deafultlevel.png)

![inspector](https://github.com/mastya6/Workshop-3/blob/main/inspectordef.png)


## Задание 3
### Применение API и гугл таблиц

Для питона я реализовала визуализацию через google sheets
```python
import gspread
import pandas
google = gspread.service_account(filename="dragonbalance-aa0b15bb83af.json")
sh = google.open("DragonPicker").sheet1

speed = 'B1:K1'
egg_drop = 'B4:K4'
chance = 'B7:K7'
level = 'B2:K2'

speed_row = sh.get(speed)[0]
egg_appearance_row = sh.get(egg_drop)[0]
direction_change_row = sh.get(chance)[0]
level_row = sh.get(level)[0]

speed_values = " ".join([str(float(num.replace(",", "."))) for num in speed_row])
egg_appearance_values = " ".join([str(float(num.replace(",", "."))) for num in egg_appearance_row])
direction_change_values = " ".join([f"{round(float(num.replace(',', '.')) * 10, 2)}%" for num in direction_change_row])
level_values = " ".join(level_row)

print(f"Level : {level_values}")
print(f"Speed : {speed_values}")
print(f"Egg Appearance Time : {egg_appearance_values}")
print(f"Chance of Direction Change : {direction_change_values}")
```
![python](https://github.com/mastya6/Workshop-3/blob/main/python.png)

А для изменения сложности уровней в Unity я изменила скрипт EnemyDragon,
который теперь берет данные из гугл таблицы, и в зависимости от выбранного 
в инспекторе уровня берет сложности из списков сложностей
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using Newtonsoft.Json.Linq;
using System.Globalization;

public class EnemyDragon : MonoBehaviour
{
    private string apiKey = "AIzaSyAGzhQb43xkMKzd1lQKZHa7vWYdzRS5oP4";
    private string sheetId = "1igOtFvgmFzXeKFV2k2Q61oOH7Ey9yvQpD4lkhAH_5dA";
    
    public int level = 1;
    public GameObject dragonEggPrefab;
    private List<float> speeds = new List<float>();
    private List<float> eggDropTimes = new List<float>();
    private List<float> directionChangeChances = new List<float>();

    private float speed;
    private float timeBetweenEggDrops;
    private float chanceDirection;
    public float leftRightDistance = 10f;

    private void Start()
    {
        StartCoroutine(InitializeDragonSettings());
    }

    private IEnumerator InitializeDragonSettings()
    {
        yield return StartCoroutine(LoadSheetData());
        speed = speeds[level - 1];
        timeBetweenEggDrops = eggDropTimes[level - 1];
        chanceDirection = directionChangeChances[level - 1];
        Debug.Log($"Dragon Settings for Level {level}:");
        Debug.Log($"Speed: {speed}");
        Debug.Log($"Time Between Egg Drops: {timeBetweenEggDrops}");
        Debug.Log($"Chance to Change Direction: {chanceDirection}%");
        Invoke("DropEgg", 2f);
    }

    private IEnumerator LoadSheetData()
    {
        yield return StartCoroutine(GetRowData("B1:K1", speeds));
        yield return StartCoroutine(GetRowData("B4:K4", eggDropTimes));
        yield return StartCoroutine(GetRowData("B7:K7", directionChangeChances));
    }

    private IEnumerator GetRowData(string range, List<float> list)
    {
        string url = $"https://sheets.googleapis.com/v4/spreadsheets/{sheetId}/values/{range}?key={apiKey}";

        using (UnityWebRequest request = UnityWebRequest.Get(url))
        {
            yield return request.SendWebRequest();

            if (request.result == UnityWebRequest.Result.Success)
            {
                ParseRowData(request.downloadHandler.text, list);
            }
            else
            {
                Debug.LogError($"Ошибка загрузки данных: {request.error}");
            }
        }
    }

    private void ParseRowData(string jsonData, List<float> list)
{
    JObject data = JObject.Parse(jsonData);
    JArray values = (JArray)data["values"];

    foreach (var value in values[0])
    {
        // Заменяем запятые на точки
        string normalizedValue = value.ToString().Replace(",", ".");
        
        // Пытаемся преобразовать строку в число
        if (float.TryParse(normalizedValue, NumberStyles.Float, CultureInfo.InvariantCulture, out float result))
        {
            list.Add(result);
        }
        else
        {
            Debug.LogWarning($"Не удалось преобразовать значение '{value}' в float");
        }
    }
}


    private void DropEgg()
    {
        Vector3 myVector = new Vector3(0.0f, 5.0f, 0.0f);
        GameObject egg = Instantiate(dragonEggPrefab);
        egg.transform.position = transform.position + myVector;
        Invoke("DropEgg", timeBetweenEggDrops);
    }

    private void Update()
    {
        Vector3 pos = transform.position;
        pos.x += speed * Time.deltaTime;
        transform.position = pos;

        if (pos.x < -leftRightDistance)
        {
            speed = Mathf.Abs(speed);
        }
        else if (pos.x > leftRightDistance)
        {
            speed = -Mathf.Abs(speed);
        }
    }

    private void FixedUpdate()
    {
        if (Random.value < chanceDirection)
        {
            speed *= -1;
        }
    }
}
```

![level](https://github.com/mastya6/Workshop-3/blob/main/changedlevel.png)
![inspector](https://github.com/mastya6/Workshop-3/blob/main/inspectorchg.png)

## Выводы

Была изучена структура проекта, проанализирована сложность и этапы ее изменения,
на их основе построена модель изменения сложностей и построена ее визуализация, 
применена на практике

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**

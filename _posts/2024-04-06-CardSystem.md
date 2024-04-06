---
layout: post
title: 카드게임 시스템 만들기
---

이번에 개발해야하는 내용은 하스스톤과 유사한 카드게임을 만드는 것이었다.
카드의 종류는 몬스터, 마법, 도깨비 카드가 있고, 카드에 있는 데이터는 구글 스프레드 시트에 다음과 같이 저장되었다.

![스프레드시트이미지](/assets/img/CardSystemDocs1.png)

스프레드 시트에는 `https://docs.google.com/spreadsheets/d/~~~~~~~/edit#gid=~~~~~` 이런식으로 저장이 되는데 /edit 전부분까지 가져온다.
`https://docs.google.com/spreadsheets/d/~~~~~~~/`부분 뒤에 `export?format=tsv&range=A2:G28"`를 붙여주면 되는데 range부분에 스프레드시트의 가져올 범위를 넣으면 된다.

최종적으로 `https://docs.google.com/spreadsheets/d/~~~~~~~/export?format=tsv&range=A2:G28"`이런식의 주소가 완성되고 이를 string 변수로 저장했다.

그 다음 `UnityEngine.NetWorking.UnityWebRequest.Get` 을 사용해서 내용을 가져온다.

UnityDocumentation 의 `UnityWebRequest.Get` 예제

```C#
using UnityEngine;
using UnityEngine.Networking;
using System.Collections;

// UnityWebRequest.Get example

// Access a website and use UnityWebRequest.Get to download a page.
// Also try to download a non-existing page. Display the error.

public class Example : MonoBehaviour
{
    void Start()
    {
        // A correct website page.
        StartCoroutine(GetRequest("https://www.example.com"));

        // A non-existing page.
        StartCoroutine(GetRequest("https://error.html"));
    }

    IEnumerator GetRequest(string uri)
    {
        using (UnityWebRequest webRequest = UnityWebRequest.Get(uri))
        {
            // Request and wait for the desired page.
            yield return webRequest.SendWebRequest();

            string[] pages = uri.Split('/');
            int page = pages.Length - 1;

            switch (webRequest.result)
            {
                case UnityWebRequest.Result.ConnectionError:
                case UnityWebRequest.Result.DataProcessingError:
                    Debug.LogError(pages[page] + ": Error: " + webRequest.error);
                    break;
                case UnityWebRequest.Result.ProtocolError:
                    Debug.LogError(pages[page] + ": HTTP Error: " + webRequest.error);
                    break;
                case UnityWebRequest.Result.Success:
                    Debug.Log(pages[page] + ":\nReceived: " + webRequest.downloadHandler.text);
                    break;
            }
        }
    }
}
```
페이지는 `/`로 구분되고, 행은 `\n`, 열은 `\t`로 구분된다.

받아올 카드 데이터를 저장하기 위해 CardData 클래스를 만들었다. 카드 종류를 구분하기 위해서 CardType 열거형도 만들어 주었다.
```C#
public enum CardType
{
    Minion,
    Spell,
    Goblin
}

public class CardData
{
    Dictionary<string, CardType> cardTypeTable = new Dictionary<string, CardType>()
    {
        {"몬스터", CardType.Minion },
        {"마법",CardType.Spell },
        {"도깨비",CardType.Goblin }
    };
    public string cardName;
    public Sprite cardImage;
    public string cardEffectComment;
    public int cardCost;
    public CardType cardType;
    public int cardAttackPower;
    public int cardHp;
    public string cardComment;

    public CardData(string[] data)
    {
        cardName = data[0];
        cardEffectComment = data[1];
        cardCost = int.Parse(data[2]);
        cardType = cardTypeTable[data[3]];
        if (cardType == CardType.Spell)
        {
            cardAttackPower = 0;
            cardHp = 0;
        }
        else
        {
            cardAttackPower = int.Parse(data[4]);
            cardHp = int.Parse(data[5]);
        }
        cardComment = data[6];
    }
}
```
CardData의 생성자에서 문자열을 받아서 CardType을 판단해주기 위해서 딕셔너리를 사용했다.

```c#
public class GoogleSheetLoader : MonoBehaviour
{
    const string googleSheetLink = "https://docs.google.com/spreadsheets/d/1VRNXgP3NavdzDkDmx-uT3b76DH0pErLPy4qieNuj22w/export?format=tsv&range=A2:G28";
    public List<CardData> cardList;

    void Awake()
    {
        if (cardList==null)
        {
            cardList = new List<CardData>();
        }
    }
    // Start is called before the first frame update
    IEnumerator Start()
    {
        using (UnityWebRequest www = UnityWebRequest.Get(googleSheetLink))
        {
          yield return www.SendWebRequest();
          if (www.isDone)
          {
              string sheetData = www.downloadHandler.text;
              ListUpDatas(sheetData);
          }
        }
    }

    private void ListUpDatas(string data)
    {
        string[] rows = data.Split('\n');
        foreach(string row in rows)
        {
            string[] columns = row.Split('\t');
            CardData card = new CardData(columns);
            cardList.Add(card);
        }

    }
    // Update is called once per frame
    void Update()
    {
        
    }
}
```
`ListUpDatas`에서 행을 구분하고, 각 열을 구분해서 CardData클래스에 넣어준다음 리스트에 추가해줬다.

![유니티이미지](/assets/img/CardSystemDocs2.png)

유니티 화면에서도 잘 나오는 모습이다.

다음에는 시트에서 이미지 가져오기 기능을 개발할 예정입니다.

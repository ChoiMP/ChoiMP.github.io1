---
title: "Welcome to Jekyll!"
date: 2024-04-06 22:12:00 -0400
categories: jekyll update
---

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;

public enum CardType
{
    Minion,
    Spell,
    Goblin
}

[System.Serializable]
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

    public CardData(string name, string effectComment, int cost, CardType type, int AttackPower, int hp, string comment)
    {
        cardName = name;
        cardEffectComment = effectComment;
        cardCost = cost;
        cardType = type;
        cardAttackPower = AttackPower;
        cardHp = hp;
        cardComment = comment;
    }

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
        using UnityWebRequest www = UnityWebRequest.Get(googleSheetLink);
        yield return www.SendWebRequest();
        if (www.isDone)
        {
            string sheetData = www.downloadHandler.text;
            ListUpDatas(sheetData);
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

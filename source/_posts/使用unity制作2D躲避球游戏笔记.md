---
title: 使用unity制作2D躲避球游戏笔记
date: 2017-04-01 21:55:56
tags:
- Unity
- C#
categories:
- 技术
---
# 游戏要求

1. 每隔2秒，随机从天空的某个位置掉落
2. 球体掉落后有弹性，再次弹起
3. 玩家控制小人可以左右移动，也可以跳跃
4. 球的属性随机，颜色随机
5. 死亡后弹出对话框
6. 碰到红色球玩家的速度会得到提升
7. 碰到黄球玩家体力-1
8. 碰到绿球玩家体力+1
9. 玩家体力为0时，GG
<!--more-->
# 建立项目

## 先建立目录结构

- Assets
    - Materials
    - Prefabs
    - Scences
    - Scripts
    - Sprites

![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/2DUnityGame/matrials.png)
![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/2DUnityGame/prefabs.png)
![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/2DUnityGame/scene_game.png)
![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/2DUnityGame/scene_loose.png)
![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/2DUnityGame/scene_start.png)
![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/2DUnityGame/scenes.png)
![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/2DUnityGame/scripts.png)
![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/2DUnityGame/sprites.png)

建立如图所示的目录结构。

## 创建场景跳转脚本

### 在script文件夹下创建LoadLevel.cs的脚本文件

```c#

using UnityEngine;
using System.Collections;

public class LoadLevel : MonoBehaviour {

    public void loadLevel(string name) {
        Application.LoadLevel(name);
    }
    public void quitRequest() {
        Application.Quit();
    }
}
```

公开一个`loadLevel()`方法，传入一个字符串参数`name`,用来访问要切换的场景的名字，方法中使用`Application.LoadLevel(name)`方法来读取一个场景。

然后继续公开一个`quitRequest()`方法，在方法中调用`Application.Quit();`来退出场景。

### 在unity中调用LoadLevel.cs的脚本

创建名为LevelManager的空物体，并把`LoadLevel.cs`挂到此物体上。

创建一个名为Start的button，在Inspector上的on click()工具栏中选择LoadLevel.loadLevel方法，并在输入框中输入Game，代表进入名为Game的scene。

![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/2DUnityGame/button_onclick.png)

这样，当点击start时就可以跳转到游戏场景。

## 游戏场景

![](https://raw.githubusercontent.com/charexcalibur/pic_markdown/master/2DUnityGame/game_scene.png)

游戏场景里主要需要以下组件：

- person: 需要添加Sprite Renderer、BoxCollider2D、Rigidbody2D和一个Person.cs脚本
- PlayRange: 下需要添加三个带BoxCollider2D的空物体，以及Border.cs脚本
- Ball: 需要添加ball.cs脚本

### Person.cs脚本的编写

```c#

using UnityEngine;
using UnityEngine.UI;
using System.Collections;

public class Person : MonoBehaviour {

    private float speed;
    private int life;
    private LoadLevel levelmanager;
    public Text lifeval;
	// Use this for initialization
	void Start () {
        life = 3;
        speed = 0.06f;
        lifeval.text = life.ToString();
        levelmanager = GameObject.FindObjectOfType<LoadLevel>();
	}

	// Update is called once per frame
	void Update () {
        MovePerson();
    }

    void MovePerson()
    {
        if (Input.GetKey(KeyCode.A))
        {
            transform.position = new Vector3(transform.position.x-speed,transform.position.y,transform.position.z);
        }
        else if (Input.GetKey(KeyCode.D))
        {
            transform.position = new Vector3(transform.position.x + speed, transform.position.y, transform.position.z);
        }
        if (Input.GetKey(KeyCode.Space))
        {
            transform.position = new Vector3(transform.position.x, transform.position.y + speed, transform.position.z);
        }
        transform.position = new Vector3(Mathf.Clamp(transform.position.x, -3.5f, 3.5f), Mathf.Clamp(transform.position.y, -2.5f, 2f), transform.position.z);
    }

    void OnCollisionEnter2D(Collision2D col)
    {
        if (col.collider.tag == "Ball")
        {
            if (col.collider.GetComponent<SpriteRenderer>().color == Color.green)
            {
                life++;
            }
            else if (col.collider.GetComponent<SpriteRenderer>().color == Color.red)
            {
                speed += 0.01f;
                speed = Mathf.Clamp(speed,0.06f,0.2f);
            }
            else if (col.collider.GetComponent<SpriteRenderer>().color == Color.yellow)
            {
                life--;
                if (life == 0)
                {
                    levelmanager.loadLevel("Lose");
                }
            }
            lifeval.text = life.ToString();
            Destroy(col.collider.gameObject);
        }

    }
}
```

Person.cs主要通过在`void Update(){}`中调用`MovePerson()`方法来控制人物的移动，通过`OnCollisionEnter2D(Collision2D col)`实现当碰撞时，根据tag来识别ball，当tag为ball时，判断`col.collider.GetComponent<SpriteRenderer>().color`从而实现生命值、速度的变化。

### Border.cs

```c#

using UnityEngine;
using System.Collections;

public class Border : MonoBehaviour {

    void OnTriggerEnter2D(Collider2D col)
    {
        if (col.tag == "Ball")
        {
            Destroy(col.gameObject);
        }
    }
}
```


当球碰到墙壁时销毁。


### Ball.cs

```c#

using UnityEngine;
using System.Collections;

public class Ball : MonoBehaviour {

    public GameObject ball0;
    private GameObject ball;
    private float m_time;
    private Color[] colorindex = { Color.yellow, Color.red, Color.green };
	// Use this for initialization
	void Start () {
        m_time = 0;
	}

	// Update is called once per frame
	void Update () {
        GenerateBall();
    }

    void GenerateBall()
    {
        if (Time.time - m_time >= 2)
        {
            ball = (GameObject)Instantiate(ball0, new Vector3(Random.Range(-3.5f, 3.5f), 2.0f+ Random.Range(-0.5f, 0.5f), 0), Quaternion.identity);
            ball.GetComponent<Rigidbody2D>().velocity = new Vector2(Random.Range(-3.5f, 3.5f), 0);
            ball.GetComponent<SpriteRenderer>().color = colorindex[(int)Mathf.Floor(Random.Range(0f,3f))];
            m_time = Time.time;
        }
    }
}
```





# go oop examples

## simple example

[Structs instead of class](https://golangbot.com/structs-instead-of-classes/)

```bash
./
    employee
        employee.go
    main.go
```

```go
//employee.go
package employee

import (
	"fmt"
)

type employee struct {
	firstName   string
	lastName    string
	totalLeaves int
	leavesTaken int
}

func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {
	e := employee {firstName, lastName, totalLeave, leavesTaken}
	return e
}

func (e employee) LeavesRemaining() {
	fmt.Printf("%s %s has %d leaves remaining", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```

```go
//main.go
package main

import "./employee"

func main() {
	e := employee.New("Sam", "Adolf", 30, 20)
	e.LeavesRemaining()
}
```

函数必须第一个字母大写，否则无法识别；

## FPS game

struct作为参数，必须作为指针才能修改;

```bash
main.go
entity/
	player.go
	gun.go
	clip.go
	bullet.go
````

```go
//main.go
package main

import(
	"fmt"
	"./entity"
	"time"
)

func main() {
	player1:=entity.NewPlayer("grey")
	enermy1:=entity.NewPlayer("zombie")
	ak47:=entity.NewGun("AK47")
	clip1:=entity.NewClip(20)
	for i := 0; i < 12; i++ {
		bullet:=entity.NewBullet(33)
		player1.ReloadBullet(&clip1,&bullet)
	}
	player1.ReloadClip(&ak47,&clip1)
	player1.PickUp(&ak47)
	//fmt.Printf("%s|%s\n",player1,enermy1)
	//player1.Shoot(&enermy1)
	//fmt.Printf("%s|%s",player1,enermy1)
	for i := 0; i < 6; i++ {
		fmt.Printf("%s|%s\n",player1,enermy1)
		player1.Shoot(&enermy1)
		time.Sleep(1000*time.Millisecond)
	}
}
```

```go
//player.go
package entity

import "fmt"

//import "fmt"

type Player struct {
	name string
	HP int
	gun *Gun
}

func NewPlayer(name string) Player {
	// similar to constructor
	return Player{name:name,HP:100,gun:nil}
}

func (player Player)ReloadBullet(clip *Clip, bullet *Bullet) {
	//bullet2clip
	clip.SaveBullet(bullet)
}

func (player Player) ReloadClip(gun *Gun, clip *Clip) {
	//clip2gun
	gun.SaveClip(clip)
}

func (player *Player) PickUp(gun *Gun) {
	player.gun=gun
}

func (player Player) String() string {
	if player.HP > 0 {
		if player.gun != nil {
			return fmt.Sprintf("%s's HP=%d,gun=%s",player.name,player.HP,player.gun)
		}else {
			return fmt.Sprintf("%s's HP=%d,without gun",player.name,player.HP)
		}
	}else {
		return fmt.Sprintf("%s is dead",player.name)
	}
}

func (player Player) Shoot(player2 *Player) {
	player.gun.Fire(player2)
}

func (player *Player) DropHP(damage int) {
	player.HP-=damage
}
```

```go
//gun.go
package entity

import "fmt"

type Gun struct {
	name string
	clip *Clip
}

func NewGun(name string) Gun {
	return Gun{name: name, clip: nil}
}

func (gun *Gun) SaveClip(clip *Clip) {
	gun.clip=clip
}

func (gun Gun) String() string {
	if gun.clip != nil {
		return fmt.Sprintf("%s,%s",gun.name,gun.clip)
	}else {
		return fmt.Sprintf("%s,without clip",gun.name)
	}
}

func (gun Gun) Fire(player *Player) {
	bullet:=gun.clip.PopBullet()
	if bullet!=nil {
		bullet.Hit(player)
	}else {
		fmt.Printf("no bullet left in clip")
	}
}
```

```go
//clip.go
package entity

import "fmt"

type Clip struct {
	max_bullets int
	bulletStack []*Bullet
}

func NewClip(max_bullets int) Clip {
	return Clip{max_bullets:max_bullets,bulletStack:nil}
}

func (clip *Clip) SaveBullet(bullet *Bullet) {
	clip.bulletStack=append(clip.bulletStack, bullet)
}

func (clip Clip) String() string {
	return fmt.Sprintf("clip=%d/%d",len(clip.bulletStack),clip.max_bullets)
}

func (clip *Clip) PopBullet() *Bullet {
	length:=len(clip.bulletStack)
	if length != 0 {
		bullet:=clip.bulletStack[length-1]
		clip.bulletStack=clip.bulletStack[:length-1]
		return bullet
	}else {
		return nil
	}
}
```

```go
//bullet.go
package entity

type Bullet struct {
	damage int
}

func NewBullet(damage int) Bullet {
	return Bullet{damage:damage}
}

func(bullet Bullet) Hit(player *Player) {
	player.DropHP(bullet.damage)
}
```

```bash
#output
grey's HP=100,gun=AK47,clip=12/20|zombie's HP=100,without gun
grey's HP=100,gun=AK47,clip=11/20|zombie's HP=67,without gun
grey's HP=100,gun=AK47,clip=10/20|zombie's HP=34,without gun
grey's HP=100,gun=AK47,clip=9/20|zombie's HP=1,without gun
grey's HP=100,gun=AK47,clip=8/20|zombie is dead
grey's HP=100,gun=AK47,clip=7/20|zombie is dead
```
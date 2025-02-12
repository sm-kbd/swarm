# `codrone_edu.drone`ライブラリにバグがあるので、`Swarm`を利用するときは`asyncio.sleep`を`original_asyncio_sleep`として宣言してください。
```python
import asyncio

from codrone_edu.drone import *

from swarm import Swarm

asyncio.sleep = original_asyncio_sleep
```
## 説明
`codrone_edu.drone`の中にある：
```python
...
original_asyncio_sleep = asyncio.sleep

async def interruptible_sleep(duration):
  interval = 0.1
  elapsed = 0.0

  while elapsed < duration:
    await original_asyncio_sleep(min(interval, duration - elapsed))
    checkInterrupt()
    elapsed += interval

asyncio.sleep = interruptible_sleep
...
```
のせいで、`asyncio.sleep`が本来の`asyncio.sleep`と異なる関数に変えられています。本来の`asyncio.sleep`が`original_asyncio_sleep`として保存されているので、`asyncio.sleep = original_asyncio_sleep`で`asyncio.sleep`を本来の関数に戻せます。

# swarm
asyncioを使ったスウォームクラス。

# 使い方  
- ドローンに使われる関数のすべてをSwarmからも使うことができます。
  - `swarm.関数()`でスウォームに入っているすべてのドローンにその関数を呼ぶことができます。
  - すべての関数に`drones=[ドローンのリスト]`で、呼ばれたいドローンのリストを引数として送ると、そのリストのドローンだけにその関数が呼ばれます。  
例:
```python
swarm.move_forward(10, "cm", 1) # スウォームのすべてのドローンを10センチ前進させる
swarm.move_forward(10, "cm", 1, drones=[drone1, drone4, drone10]) # スウォームの中からdrone1、drone4、とdrone10だけを前進させる
```
- asyncなので`time.sleep()`ではなく`asyncio.sleep()`を使ってください。
```python
swarm.move_forward(10, "cm", 1)
await asyncio.sleep(1)
```
※ `time.sleep()`使うと同時飛行ではなくなる可能性もあります。  
※ もし`codrone_edu.drone.Drone`の関数で`time.sleep`を使っているせいで同時飛行を妨げるものがあれば教えてください。可能なら`async`として再現してみます。
- ドローンを指定するにはポート名も使用できます。
```python
swarm.move_forward(10, "cm", 1, drones=[drone1, drone2, drone3]) # ドローンオブジェクト指定
swarm.move_forward(10, "cm", 1, drones=["port3", "port3", "port3"]) # ポート名指定
```
# 例
```python
import asyncio

from codrone_edu.drone import *

from swarm import Swarm

async def main():
  swarm = Swarm()
  drone1 = Drone()
  drone2 = Drone()
  drone3 = Drone()

  swarm.add_and_pair("port1", drone1, "port2", drone2, "port3", drone3)
  await swarm.takeoff()
  swarm.move_forward(...)
  await asyncio.sleep(...)
  await swarm.land()
  swarm.close()

asyncio.run(main())
```

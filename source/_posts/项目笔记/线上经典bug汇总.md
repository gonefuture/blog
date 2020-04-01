# 线上经典bug汇总

## 原子变量的`incrementAndGet()`和`get()`连用造成的并发问题

这是一段关于秘境玩法的业务代码，是在控制秘境这中个人副本中怪物和boss的刷新。
秘境的玩法：进入副本后，第一波小怪开始刷新，玩家击杀小怪，得到能量值。一波小怪死亡刷新下一波小怪，直到能量值达到指定数，开始刷新最终boss。杀死boss，玩法结束，获得奖励。

### 出现bug的代码

下面为开发版本修复后的代码

```java
// 官阶幻境场景脚本
public class MysterySceneObserver extends WorldWorldMapInstanceObserverAdapter {

    // 能量
    private AtomicInteger energy = new AtomicInteger();

    // 当前场景小怪数量
    private AtomicInteger monsterNum = new AtomicInteger();

    // ....... 省略....

    @Overrdide
    public void objectEnterMap(AbstractVisibleObject object, WorldMapInstance mapInstance) {
        if(object instanceof Boss) {
            // ....... 省略....
        } else if (object instancof Monster) {
            monsterNum.incrementAndGet();
            ((Monster) object).getObserveController().attachWithCurrentMap(ICreatureDead.class, (monsterDead, lastAttacker -> {
                // 原本没有修复前这里并没有`monsterLeftNum`这个参数，而是直接执行`decrementAndGet()`就算了，并未赋值变量
                int monsterLeftNum = monsterNum.decrementAndGet();
                if(mysterySceneResource.getMonsterEnergy().contailnsKey(monsterDead.getObjectKey())) {
                    int addEnergy = mysterySceneResource.getMonsterEnergy().get(monsterDead.getObjectKey());
                    energy.addAndGet(addEnergy);
                    if(!refreshBoss && status == SceneStatus.INIT) {
                        refreshMonster(monsterLeftNum, mapInstance);
                    }
                    // 发送当前能量
                    mapInstance.playerIterator().forEachRemaining();
                    mapInstance.playerIterator().forEachRemaining(fightPlayer -> fightPlayer.sendPacket(MysterySceneEneryResp.valueOf(enery.get(), getLeftSeconds())));
                }
            }));
        }
    }


    // 判断刷怪 剩余的小怪数量
    private void refreshMonster(int monsterLeftNum, WorldMapInstance mapInstance) {
        // 攒满能量
        if(energy.get() >= mysterySceneResource.getMaxEnergy()) {
            refreshBoss = true;
            energy.getAndSet(mySterySceneResoure.getMaxEnergy());
            // 清除小怪
            mapInstance.asyncClearObject(visibleObject -> visibleObjec instanceof Monster && !(visibleObject instanceof Boss));
            // 召唤boss
            SpawnManger.getInstance().spawnAllNpc(mapInstance, mysterySceneResource.getBossLayerIndex(), spawnResource -> {
                if(spawnResource.size() == 1) {
                    return spawnResources;
                }
                List<SpawnResource> randomList = new ArrayList<>(spawnResource);
                int index = RandomUtils.betweenInt(0,randomList.size()-1, true);
                List<SpawnResource> result = new  ArrayList<>();
                result.add(randomList.get(index));
                return result;
                );
            });
        } else {
            // 能量未召唤boss继续刷小怪
            /**
            *  这里未修复之前是`if(monsterNum.get()== 0)`，
            **/
            if(monsterLeftNum == 0) {
                if(refreshMonsterIndex < mysterySceneResource.getMonsterIndexList().size()) {
                    // 召唤小怪波次
                    SpawnManager.getInstance().spawnAllNpc(mapInstance, mysterySceneResource.getMonsterIndexList().get(refreshMonsterIndex), null);
                }
                refreshMonsterIndex++;
            }
        }
    }
}
```

### 解析

由于怪物死亡的事件是异步执行的，当玩家同时杀死多个怪物时，就可能出现一种情况：当某个线程刚执行原子变量`monsterNum`的`decrementAndGet()`方法并且尚未执行`get()方法`时，另一个线程可能执行了`decrementAndGet()`方法，导致前者线程中`monsterNum`不正确。

### 解决方案

* 线上版本的热修: 用`synchronize(this)`包裹`decrementAndGet()`方法和`get()`方法的调用。
* 开发版本的修复：为`refreshMonster`方法增加一个参数`int monsterLeftNum`，由`decrementAndGet()`直接赋值，在`refreshMonster`内不再使用`if(monsterNum.get()== 0)`，而是改为`if(monsterLeftNum == 0)`

### 总结

* 在实际开发中，对原子变量的使用必须要小心，不要以为原子变量就不会出现并发问题。
* 原子变量的`decrementAndGet()`和`get()`方法连续使用时，一定要注意`get()`方法求出的值是够有可能已经改变
* 上诉代码中两个原子变量`AtomicInteger energy`和`AtomicInteger monsterNum`的同时操作和参与判断，也是很有可能有并发问题的，这要根据业务场景来处理。
  

## 修改配置类
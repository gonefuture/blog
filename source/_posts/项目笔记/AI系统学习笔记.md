# AI系统学习笔记

## AbstractAi

所有AI都继承于`AbstractAi`，他拥有一个AI所必须的状态迁移器，基础状态处理器。

```java
public abstract class AbstractAi {
    // 状态迁移器
    private IStateConverter stateConverter;

    // 基础状态处理器
    private AbstractBaseStateHandler baseStateHandler;

    // AI事件处理器集合
    private Map<AiEvent, AbstractEventHandler> eventHandler = new HashMap<>();

    // AI状态处理器集合
    private Map<AIState, AbstractStateHandler> stateHandlers = new HashMap<>();

    protected Npc owner;

    protected AIState aiState = AIState.NONE;

    // 执行帧数，用于控制各个状态处理器的执行频率
    private long tickCount;

    // AI配置信息
    private AiResource aiResource;

    private SkillSelector skillSelector;

    // Npc外显状态，分为“外显激活”和“外显未激活”两种。
    // 大部分的NP都是“外显激活”状态，只有个别特殊怪会有“外显未激活”状态，如封印僧侣、食人花等。
    private NpcShowStatus npcShowStatus = NpcShowStatus.SHOW_ACTIVED;

    // 延迟处理AI事件
    // 从AI线程以外的线程抛出的事件应该先添加到AI线程中，然后延时处理

    // owner of this AI
    public Npc getOwner() {
        return ownwer;
    }

    // 装配事件处理器

    // 装配状态处理器

    
    // 心跳
    private void tick() {
        this.tickCount++;
        // 处理被动技能
        handlerPsvSkill();

        // 处理基础行为
        if(baseStateHandler != null) {
            baseStateHandler.handleState(this);
        }

        // 判断状态迁移
        if(stateConverter == null) {
            throw new IllegalStateException(getClass().getSimpleName()+" 缺少状态迁移器，AI宿主："+owner.getName());
        }

        stateConverter.concerState(this);

        // 处理具体状态
        if(!owner.isActing()) {
            AbstractStateHandler stateHandler = stateHandlers.get(aiState);
            if(stateHandler == null) {
                throw new IllegalStateException(getClass().getSimpleName() + "缺少相应的状态处理器，AiState : " +aiState + ",AI宿主："+ owner.getName()));
            }
            if(this.tickCount % stateHandler.getFrequency(this) == 0) {
                stateHandler.handlerState(this);
            }
        }
    }


    //  开启心跳
    public void schedule() {
        if(!isScheduled()) {
            this.tickCount = 0;
            this.tick.trySchedule();
        }
    }
}

```
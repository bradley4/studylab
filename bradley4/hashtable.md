### Locking
- 해쉬테이블은 오브젝트 레벨의 락을 건다. 전체 맵이 락이 걸린다. 
- 컨커렌트해쉬맵은 전체 맵이 아닌 해당 엔트리에 대해서만 락이 걸린다. 
  각각의 맵은 세그먼트로 나눠지고 세그먼트는 각각 자신의 락을 가진다. 
  쓰레드가 해당 세그먼트에 접근하려면 세그먼트 락을 획득해야한다.

### HashMap
- key, value에 null 가능
- thread safe 하지 않음

### HashTable
- key, value에 null 허용안함
- thread safe하다.
- JDK 1.1, 1.2 
- deprecation
- 상대적으로 속도가 느리다.

### ConcurrentHashMap
- key, value에 null 허용안함.
- thread safe하다.

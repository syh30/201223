# Spatial data를 10,000배 잘 다루게 된 방법   

안녕하세요. 메쉬코리아 기반서비스서버팀 강성일입니다.    
이번에 Spatial data를 **10,000배!** 잘 다루게 된 방법에 대해 공유하고자 글을 쓰게 되었습니다 :smile:   

들어가기에 앞서, **메쉬코리아**는 음식, 화장품, 편의점 상품과 같은 상품을 실시간으로 배달하는 서비스인 **부릉(Vroong)**:articulated_lorry:을 운영하는 회사인데요!    
배달 주문을 처리하기 위해서는    
+	상점과 고객 사이의 거리와 장애물 등을 고려하여 처리할 수 있는 주문건인가?    
+	배달 기사님께는 얼마만큼의 금액을 지불해야 하는가?
+ 상점에서는 얼마만큼의 금액을 받아내야 하는가?    

를 판단해야 합니다.    

그래서 **spatial data**를 이용하여 각 상점의 배송 가능한 권역들, 배달이 불가능한 권역들을 관리하는 권역 시스템을 운영하고 있습니다.    

권역 시스템에서는 **spatial data**를 효율적으로 다루기 위해서 **PostgreSQL**에서 지원하는 **PostGIS**를 이용하고 있는데요.   
**지도 화면에서 보이는 영역과 행정동/법정동 polygon의 intersects를 계산하고 polygon을 가져오는 과정에서 큰 성능 이슈를 가지고 있었습니다.** *(worst case에서는 조회시간이 60초 정도 걸렸습니다,,,눈물)*    

성능이 떨어지는 문제의 쿼리는 아래와 같았습니다.

```
SELECT region.*
FROM region
INNER JOIN address_polygon on region.pk=address_polygon.region_pk
INNER JOIN address on address_polygon.address_pk=address.pk
WHERE address.address_type = 'LEGAL'
AND region.region_type = 'ADDRESS_REGION'
AND region.status = 'ENABLED'
AND st_intersects(
    region.polygon,
    ST_MakeEnvelope(126.9373539, 37.5210172, 127.0540836, 37.5873607, 4326)
);
```
그럼 이 문제를 어떻게 해결했는지, 어떻게 성능을 10,000배 올릴 수 있었는지 지금부터 이야기해보겠습니다!  
###

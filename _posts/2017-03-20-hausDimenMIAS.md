---
layout: post
title:  "classifying be/mal in MIAS cases using Hausdorff dimensions"
date:   2017-03-20 22:13:00
comments: true
categories: main
---

## Box Counting Method를 이용한 Fractal 차원의 계산 및 실제 데이터 분석 사례 - MIAS 데이터 베이스를 활용한 종양 진단.


##### 1. 도입


&nbsp;&nbsp;최근들어 데이터 분석에 관한 관심이 높아지면서, 기하학에도 데이터 분석에 기하적인 개념을 활용하고자 하는 움직임이 많아지고 있다. 이 글에서는 Fractal dimension을 실제 데이터 분석에 사용하고 있는 사례를 따라갈 것이며, 그 내용이 타당한지 생각해 볼 것이다.  
&nbsp;&nbsp;Fractal dimesnion에 대한 직접적인 설명은 생략한다. 이는 [5] 및 wiki[4]에 잘 소개되어 있다. Fractal dimension을 구하는 방법 중 가장 간단하면서 잘 알려진 방법이 Box Counting Method인데 이 방법에 대해서 간략하게 언급한다. 이를 구현한 알고리즘 및 코드는 본문 내에 기재하였다.  
&nbsp;&nbsp;그리고 이 개념을 사용하여 mini-MIAS(Mammographic Image Analysis Society) 데이터베이스[2]의 이미지를 분석한 사례[5]에 대해서 생각해보고자 한다. 데이터 분석에는 실제 분석 과정 이외에도 전처리 과정이 필요하다. [5]에는 데이터 전처리 과정에 대해 설명이 되어있기는 하지만 구체적인 방법에 대한 기술은 부족하다. 전처리 과정을 실제로 따라해 보면서 어떤 결과를 얻는지에 대해 고찰해 볼 것이다. 

##### 2. Box counting method

Box Counting Method 다음과 같이 정의한다. [3]

<!--
![title](img/figure1.png)
-->
<img src="{{ site.url }}/assets/images/HausdorffMIAS/figure1.png">

\\(\epsilon\\)의 길이를 갖는 $$\mathbb{R}^{n}$$의 큐브를 이용해서 S를 덮을 때, 그 갯수와 $$\epsilon$$의 비를 의미하는 값이다. 여기서 S는 유클리드 공간 $$\mathbb{R}^{n}$$의 부분집합이며, 차원을 계산하려는 대상이다.  $$N(\epsilon)$$은 큐브의 갯수를 의미한다.

일반적으로 분석하고자 하는 데이터의 갯수는 유한하기 때문에 lim값을 그대로 사용하지는 않고 $$log(1/\epsilon)$$ - $$logN(\epsilon)$$ 그래프의 기울기를 $$dim_{box}(S)$$로 사용할 것이다. 이는 데이터의 차원이 데이터를 어느 정도의 스케일로 바라보느냐에 따라 달라진다는 사실을 고려하여 선정한 것이다.

아래는 Box counting method를 구현한 코드이다. Matlab R2015 b에서 작성하였다. 

구현한 코드는 다음과 같이 크게 두 함수로 구성되어있다.
 * 첫번째 함수는 getIndicesFixed라는 이름으로 정의한 함수이다. S를 특정한 길이를 갖는 cube로 잘랐을 때, 각각의 데이터 셋에 들어있는 데이터가 어느 cube에 속해있는지 알아내는 함수이다. 행렬형태로 표현한 dataset의 형태(shape)는 데이터의 갯수* 데이터 feature의 갯수로 표현되어야 한다. 분석하고자 하는 dataset, 데이터 feature의 갯수, 데이터가 퍼져있는 범위(range), $$\epsilon$$(cube의 한변 길이)을 입력변수로 받아서 각 데이터가 속해있는 cube의 인덱스를 행으로 갖는 행렬을 내보낸다.


```
function indicesSet=getIndicesFixed(dataset,dim,range,interval)
% return the index information which cell has a data.

%dataset : each row vector should mean a datum. / n*m matrix.
%dim : number of features of data / integer.
%range : matrix like [-1,1,-1,1,-1,1].
%interval : the size of interval making grid.
    
    %initialization
    grid=zeros(dim,1);

    for i=1:dim
        grid(i)=floor((range(2*i)-range(2*i-1))/interval)+1;
    end
   
    % for each data, specify the location of data in grids, iter = #ofdata
    iter=length(dataset(:,1));
    indicesSet=zeros(iter,dim);
    
    for l=1:iter
        testpoint=dataset(l,:);
        indexOfPoint=zeros(dim,1);

        for k=1:dim
            while range(2*k-1)+interval*(indexOfPoint(k)+1)<testpoint(k);
                indexOfPoint(k)=indexOfPoint(k)+1;
            end
            indexOfPoint(k)=indexOfPoint(k)+1;
        end
    indicesSet(l,:)=indexOfPoint';
    end
end
```

 * 두번째 함수는 plotDimByHausDim라는 이름으로 정의하였다. 이 함수는 $$log(1/\epsilon)$$ - $$logN(\epsilon)$$ 그래프를 그린다. 입력변수는 getIndicesFixed함수와 비슷하며 interval 대신에 domain을 사용한다. 여기서 domain은 $$\epsilon$$값들의 집합이다.

```
function [x,y]=plotDimByHausDim(dataset,dim,range,domain)
% plot log eps - log #ofbox has a data. and return data.

% use getindicesFixed.

% dataset = m*n matrix which m is number of data , each row = each data, n
% is number of features(dimension)
% dim=initial dim
% range should be inserted as [1dmin, 1dmax, 2dmin, 2dmax, ...]
% domain is a vector.

n=length(domain);    
boxCountingValues=zeros(n,1);

    tic
    for i=1:n
        temp=getIndicesFixed(dataset,dim,range,domain(i));
        boxCountingValues(i)=length(unique(temp,'rows'));
    end
    toc
    
    figure
    scatter(log(domain),log(boxCountingValues),1);
    xlabel('log epsilon')
    ylabel('log #boxeshavingdata')
    
    
    x=log(domain)';
    y=log(boxCountingValues);
end
```

위의 두 함수로부터 실제로 데이터의 Fractal dimension을 구하기 위해서는 다음과 같이 실행하면 된다.

1. data를 불러와서 n*m 형태의 행렬로 만든다. 데이터 하나는 한 행에 들어가야 한다.
2. domain을 분석하고자 하는 정도의 범위내에서 결정한다.
3. plotDimByHausDim함수를 실행한다.

아래는 이에 대한 예시이다.

```
%import data from python imread.
load('export010.txt')

% define a domain to use the function plotDimByHausDim
domain=linspace(0.6065,1,50);

% plot the graph means dimension
plotDimByHausDim(export010,2,[0,200,0,180],domain)



```

##### 3. mini MIAS 데이터 베이스를 이용한 실제 데이터 분석에 활용[5]

MIAS 데이터 베이스는 연구목적으로 공개된 mammography(유방촬영술, 네이버)를 모아놓은 이미지들의 데이터 집합이다. 유방촬영술이란 유방암을 진단하기 위해 하는 유방전용 x-ray 사진 촬영인데, 이 기법은 유방암을 진단하는데 가장 우수하기 때문에 많이 사용되었다.
 데이터베이스[2]의 이미지들을 살펴보면 각 이미지와 함께 label이 붙어있는데 이는 각각의 환자 상태와 종양이 있는 경우 종양의 위치, 크기를 pixel 정보로 담고 있다.  
 [5]에서는 mini-MIAS의 사진들을 분석하여, 종양이 있는 이미지들 중 종양이 악성인 경우는 box counting method로 계산한 fractal dimension 값이 양성인 경우보다 높게 나오는 경향이 있다는 결론을 내리고 있다.
 [5]가 이 사진들을 분석한 과정을 간략하게 소개한다. 우선 mini-MIAS에 있는 이미지에서 종양이 있는 이미지를 선정한다. 그리고 noise가 많이 있는 상태로 분류할 수 있는 이미지(석회화가 일어난 경우 등)를 걸러낸다. 그렇게 해서 얻어낸 11개의 양성 종양 사진과 15개의 악성 종양 사진들의 fratal dimesnion를 계산한다. fractal dimension을 구하기 위하여 가장 처음으로 이미지의 전처리를 하는데 그 전체적인 흐름은 다음과 같다.

1. 이미지에서 종양을 도드라지게 하기 위해서 Equalize한다. 아래의 왼쪽 그림이 원본, 오른쪽이 equalize한 사진이다.
<!--
![title](img/figure2.png)
-->
<img src="{{ site.url }}/assets/images/HausdorffMIAS/figure2.png">

2.이미지의 각 pixel이 하얀색과 검정색만을 가지도록 binary image 형태로 바꾼다. Area of Interest, 즉 종양이 있는 부위를 선정한다.

<!--
![title](img/figure3.png)
-->
<img src="{{ site.url }}/assets/images/HausdorffMIAS/figure3.png">

3.Segmented된 이미지의 Outline만을 추출해 낸다.[6]

<!--
![title](img/figure5.png)
-->
<img src="{{ site.url }}/assets/images/HausdorffMIAS/figure5.png">

위와 같이 전처리한 데이터들의 Fractal dimension을 구한 결과물이 아래의 표이다.

<!--
![title](img/figure6.png)
-->
<img src="{{ site.url }}/assets/images/HausdorffMIAS/figure6.png">

표를 보면 악성인 경우 대부분 fractal dimension 값이 상대적으로 양성인 경우에 비해서 높게 나왔다는 것을 확인 할 수 있다.

##### 4. Article 비판

[5]에서는 위와 같이 전처리한 데이터들의 Fractal dimension의 크기를 보면 악성인 경우 2정도의 값이 나타나는 반면, 양성 종양은 1.2에서 1.7정도로 상대적으로 낮은 값이 나타나는 경향이 있다고 결론짓고 있다. 이를 통해 fractal dimension이 유방의 종양의 양성, 음성을 구분하는 효율적인 Classifier가 된다고 말한다.
 

하지만 여기서 악성 종양의 데이터가 거의 2차원에 가까운 값을 갖는다는 것은 조금 문제가 있어 보인다. 전처리한 사진은 Segment된 종양의 둘레이기 때문에 거의 1차원에 가까운 형태가 될 것이다. 이에 착안하여 계산이 제대로 되지 않았다고 생각하고 처음부터 다시 계산을 해보았다.

데이터 처리 과정은 다음과 같다. 이미지 전처리 과정에서 Photoshop CS6을 사용하였으며, 전처리된 이미지를 숫자로 이루어진 행렬로 바꾸어 처리하기 위해 Python의 matplotlib를 사용하였다. Fractal Dimension는 Matlab에서 계산하였다.

아래 그림은 mdb028(악성 종양)의 전처리 과정을 나타낸다.

<!--
![title](img/figure9.png)
-->
<img src="{{ site.url }}/assets/images/HausdorffMIAS/figure9.png">

아래의 그림은 mdb028의 $$log(1/\epsilon)$$ - $$logN(\epsilon)$$ 그래프이다.

<!--
![title](img/figure10.png)
-->
<img src="{{ site.url }}/assets/images/HausdorffMIAS/figure10.png">

아래 그림은 mdb069(양성 종양)의 전처리 과정을 나타낸다.

<!--
![title](img/figure8.png)
-->
<img src="{{ site.url }}/assets/images/HausdorffMIAS/figure8.png">


아래의 그림은 mdb069의 $$log(1/\epsilon)$$ - $$logN(\epsilon)$$ 그래프이다.

<!--
![title](img/figure11.png)
-->

<img src="{{ site.url }}/assets/images/HausdorffMIAS/figure8.png">

위와 같은 과정을 반복하여 9개의 sample(악성 3개, 양성 6개)에 대해서 계산을 하였다. 그 결과물은 아래의 표와 같다.

<!--
![title](img/table1.png)
-->
<img src="{{ site.url }}/assets/images/HausdorffMIAS/table1.png">

Fractal Dimension 계산에 영향을 줄 수 있는 요소는 최대한 배제하기 위해 test example은 mini-MIAS[2]의 이미지중 CIRC M/B인 경우만을 선택하였다. 표의 FD 값은 box counting algorithm으로 계산한 fractal dimension 값인데 이는 데이터를 얼마만큼 가까이에서 보느냐에 따라 다르기 때문에 가깝게(FD1) 본 경우와 멀리서 본(FD2) 경우로 나누어서 계산을 하였다.
계산을 한 결과 양성 종양과 음성 종양의 계산값이 크게 차이 나지 않는 것을 확인하였다. 물론 비교한 데이터의 수가 많지 않은 것을 고려하였을 때 섣부르게 결론짓기는 어렵겠지만,  [5],[6]과 같이 악성 종양의 경우에 조금더 불규칙한 형태의 fractal 구조가 나타난다고 말하기는 조금 무리가 있어 보인다.  
이를 개선하기 위해서는 종양의 둘레 부분만 추출해 낼 것이 아니라, 사진에서 하얗게 보이는 부분들의 패턴이 모두 들어날 수 있도록 전처리를 해야하는게 아닐까 추측해 본다.

##### 5. 참고 문헌

1. J Suckling et al, The Mammographic Image Analysis Society Digital Mammogram Database Exerpta Medica. International Congress Series 1069 pp375-378. 1994
2. J Suckling, The mini-MIAS database of mammograms, http://peipa.essex.ac.uk/info/mias.html
3. wiki, Minkowski–Bouligand dimension, https://en.wikipedia.org/wiki/Minkowski%E2%80%93Bouligand_dimension
4. wiki, Fractal dimension, https://en.wikipedia.org/wiki/Fractal_dimension
5. sdeg junior, Eractal Dimension for Characterization of Focal Breast Lesions, 2013
6. Crisan, D.A., Dobrescu, R. ; Planinsic, P.. “Mammographic Lesions Discrimination Based on Fractal Dimension as an Indicator”. Systems, Signals and Image Processing, 2007 , pp 74-77.

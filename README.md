### Alberto Tavares de Oliveira
### Francisco Bianor Souza de Medeiros

## Sumário teste
- [Sumário teste](#sumário-teste)
  - [1 - Manipulando pixels em uma imagem](#1---manipulando-pixels-em-uma-imagem)
    - [Exercício 1.1 - regions.cpp](#exercício-11---regionscpp)
    - [Exercício 1.2 - trocaregioes.cpp](#exercício-12---trocaregioescpp)
  - [2 - Preenchendo regiões](#2---preenchendo-regiões)
    - [Exercício 2.1 - labelingRGB.cpp](#exercício-21---labelingrgbcpp)
    - [Exercício 2.2 - labelingholes.cpp](#exercício-22---labelingholescpp)
  - [3 - Manipulação de histogramas](#3---manipulação-de-histogramas)
    - [Exercício 3.1 - equalize.cpp](#exercício-31---equalizecpp)
    - [Exercício 3.2 - motiondetector.cpp](#exercício-32---motiondetectorcpp)
  - [4 - Filtragem no domínio espacial i](#4---filtragem-no-domínio-espacial-i)
    - [Exercício 4.1 - laplgauss.cpp](#exercício-41---laplgausscpp)
  - [5 - Filtragem no domínio espacial ii](#5---filtragem-no-domínio-espacial-ii)
    - [Exercício 5.1 - tiltshift.cpp](#exercício-51---tiltshiftcpp)
    - [Exercício 5.2 - tiltshiftvideo.cpp](#exercício-52---tiltshiftvideocpp)
  - [Jekyll Themes](#jekyll-themes)
  - [Support or Contact](#support-or-contact)

Página dedicada às respostas dos exercicios de programação da disciplina.

### 1 - Manipulando pixels em uma imagem

#### Exercício 1.1 - regions.cpp

Imagem original:

![biel.png](https://user-images.githubusercontent.com/56025096/124983972-0a7c9480-e00f-11eb-8aae-d0994bdaf1bb.png "biel.png") 

Código [regions.cpp](https://github.com/Albertotavaresdeoliveira/PDI-UFRN/blob/gh-pages/regions.cpp):

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

int main(int, char**){
  cv::Mat image;
  cv::Point p1, p2;

  image= cv::imread("biel.png",cv::IMREAD_GRAYSCALE);
  if(!image.data)
    std::cout << "nao abriu a imagem" << std::endl;

    cout<<"Digite P1 e P2 (x1,y1,x2,y2):"<<endl;
    cin>>p1.x >> p1.y >> p2.x >> p2.y;

  cv::namedWindow("janela", cv::WINDOW_AUTOSIZE);

  for(int i=p1.x;i<p2.x;i++){
    for(int j=p1.y;j<p2.y;j++){
       image.at<uchar>(i,j) = 255 - image.at<uchar>(i,j);
    }
  }

  cv::imshow("negativo da imagem", image);
  cv::imwrite("negativo da imagem.png", image);
  cv::waitKey();
  return 0;
}
```
Imagem obtida usando os pontos P1(20, 50) e P2(200, 200):

![negativo da imagem](https://user-images.githubusercontent.com/56025096/124983322-2e8ba600-e00e-11eb-8a58-b124cd974f17.png)


#### Exercício 1.2 - trocaregioes.cpp

O código a seguir foi implementado de forma a trocar os quadrantes de uma imagem ao longo das diagonais. Foi usada a imagem **biel.png** como exemplo.

Código [trocaregioes.cpp](https://github.com/Albertotavaresdeoliveira/PDI-UFRN/blob/654cf64d89688fec9db766f6e2f2105782da475d/trocaregioes.cpp):

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

int main(int, char**){
  cv::Mat image, quadrantes;
  int width, height;

  image= cv::imread("biel.png",cv::IMREAD_GRAYSCALE);
  if(!image.data)
    std::cout << "nao abriu a imagem" << std::endl;

  cv::namedWindow("janela", cv::WINDOW_AUTOSIZE);

  quadrantes = image.clone();

  width=image.cols;
  height=image.rows;
  std::cout << width << "x" << height << std::endl;

  for(int i=0; i<height/2; i++){
    for(int j=0; j<width/2; j++){
      quadrantes.at<uchar>(i,width/2+j) = image.at<uchar>(height/2+i,j); //1° quadrante
      quadrantes.at<uchar>(i,j) = image.at<uchar>(height/2+i,width/2+j); //2° quadrante
      quadrantes.at<uchar>(height/2+i,j) = image.at<uchar>(i,width/2+j); //3° quadrante
      quadrantes.at<uchar>(height/2+i,width/2+j) = image.at<uchar>(i,j); //4° quadrante
    }
  }

  cv::imshow("imagem original", image);
  cv::imshow("quadrantes trocados", quadrantes);
  cv::imwrite("quadrantes trocados.png", quadrantes);
  cv::waitKey();
  return 0;
}
```
Imagem obtida:

![quadrantes trocados](https://user-images.githubusercontent.com/56025096/125082069-1d3daa80-e09d-11eb-855a-3207b8a69225.png)

### 2 - Preenchendo regiões

#### Exercício 2.1 - labelingRGB.cpp

Uma forma de identificar mais de 255 objetos na cena usando a rotulação é rotular os objetos com outras cores, ao invés de somente na escala de cinza. Para esse exemplo foi implementado o código abaixo.

Imagem utilizada:

![bolhas](https://user-images.githubusercontent.com/56025096/125695403-8a1ca6de-6ccf-4bbb-b1c0-709586fdadcf.png)   ![labelingRGB](https://user-images.githubusercontent.com/56025096/125695045-529364a3-5ecc-4914-bb7b-396731c7bd7b.png)

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

int main(int argc, char** argv){
  cv::Mat original, image, semobjnasbordas, realce;
  cv::Vec3b val, B, P, C;
  int width, height, nobjects = 0;

  //Branco
  B[0] = 255; B[1] = 255; B[2] = 255;
  //Preto
  P[0] = 0; P[1] = 0; P[2] = 0;
  //Cinza
  C[0] = 127; C[1] = 127; C[2] = 127;

  cv::Point p;
  //image = cv::imread(argv[1], cv::IMREAD_GRAYSCALE);
  image= cv::imread(argv[1],cv::IMREAD_COLOR);
  original = image.clone();

  if(!image.data){ std::cout << "imagem nao carregou corretamente\n"; return(-1); }

  width=image.cols; height=image.rows;
  cout << width << "x" << height << endl;

  p.x=0; p.y=0;

  //Processo para gerar imagem apenas em preto e branco (elimina tons intermediários)
  for(int i=0; i<height; i++){
    for(int j=0; j<width; j++){
      if(image.at<uchar>(i,j) >= 127) { image.at<uchar>(i,j) = 255; }
      else { image.at<uchar>(i,j) = 0; }
    }
  }

  semobjnasbordas = image.clone();

  //Removendo os objetos das 4 bordas
  for(int i=0; i<height; i++){
      if(semobjnasbordas.at<Vec3b>(i,0) == B){  p.x=0; p.y=i; cv::floodFill(semobjnasbordas,p,P); }
    }

  for(int i=0; i<height; i++){
      if(semobjnasbordas.at<Vec3b>(i,width-1) == B){ p.x=width-1; p.y=i; cv::floodFill(semobjnasbordas,p,P); }
    }
    
  for(int j=0; j<width; j++){
      if(semobjnasbordas.at<Vec3b>(0,j) == B){ p.x=j; p.y=0; cv::floodFill(semobjnasbordas,p,P); }
    }

  for(int j=0; j<width; j++){
      if(semobjnasbordas.at<Vec3b>(height-1,j) == B){ p.x=j; p.y=height-1; cv::floodFill(semobjnasbordas,p,P); }
    }

    image = semobjnasbordas.clone();

    val[0] = 0; //B
    val[1] = 0; //G
    val[2] = 0; //R

    // busca objetos presentes
  for(int i=0; i<height; i++){
    for(int j=0; j<width; j++){
      if(image.at<Vec3b>(i,j) == B){
        // achou um objeto
        p.x=j; p.y=i; nobjects++;
        val[0] = 0;
        val[1] = nobjects % 255;
        val[2] = nobjects - ((nobjects % 255) * 255);

  		  // preenche o objeto com o contador
        //cv::floodFill(image,p,cv::Scalar((val[0]), (val[1]), (val[2]), 0));
        //cv::floodFill(image,p,cv::Scalar((val[0]), (val[1]), (val[2]), 0));
        cv::floodFill(image,p,val);
      }
    }
  }

  cout << "a figura tem " << nobjects << " bolhas" << endl;
  //cv::equalizeHist(image, realce);
  cv::imshow("imagem original clone", original);
  cv::imshow("image", image);
  cv::imshow("sem objetos que tocam as bordas", semobjnasbordas);
  cv::imwrite("labelingRGB.png", image);
  cv::waitKey();
  return 0;
}
```

#### Exercício 2.2 - labelingholes.cpp

O segundo código foi implementado para a rotulação de regiões e contagem de objetos, que não tocam na borda da imagem, com e sem buracos internos. Para o algorítimo foi previsto também que objetos com mais de um buraco também podem existir. 

Sequência do processamento da imagem:

![bolhas2buracos](https://user-images.githubusercontent.com/56025096/125688922-2fa6f0f9-b672-416b-a1c7-cf7fe9c7365a.png)   ![semobjnasbordas](https://user-images.githubusercontent.com/56025096/125699222-f8f1c3d2-3be5-470b-9c54-383db962f55c.png)   ![labeling](https://user-images.githubusercontent.com/56025096/125699305-09be40c6-1b2f-4e40-9d9d-ac9e2e5ec992.png)

![corfundoalterado](https://user-images.githubusercontent.com/56025096/125700096-5b16bcac-c760-4698-931f-5b0c81dc29b3.png)   ![output](https://user-images.githubusercontent.com/56025096/125699930-ed34b79b-2718-4cd2-8641-5b5d8c482f4d.png)


Abaixo é mostrado a saída no terminal de comando:

![Captura de tela de 2021-07-14 17-26-01](https://user-images.githubusercontent.com/56025096/125688485-3642f9b5-4e35-41b5-b4d9-7296ad32c969.png) 

Código [labelingholes.cpp]():

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace cv;

int main(int argc, char** argv){
  cv::Mat original, image, semobjnasbordas, realce;
  int width, height, nobjects = 0, objcomburacos = 0;

  cv::Point p;
  image = cv::imread(argv[1], cv::IMREAD_GRAYSCALE);
  original = image.clone();

  if(!image.data){ std::cout << "imagem nao carregou corretamente\n"; return(-1); }

  width=image.cols; height=image.rows;
  std::cout << width << "x" << height << std::endl;

  p.x=0; p.y=0;

  //Processo para gerar imagem apenas em preto e branco (elimina tons intermediários)
  for(int i=0; i<height; i++){
    for(int j=0; j<width; j++){
      if(image.at<uchar>(i,j) >= 127) { image.at<uchar>(i,j) = 255; }
      else { image.at<uchar>(i,j) = 0; }
    }
  }

  semobjnasbordas = image.clone();

  //Removendo os objetos das 4 bordas
  for(int i=0; i<height; i++){
      if(semobjnasbordas.at<uchar>(i,0) == 255){  p.x=0; p.y=i; cv::floodFill(semobjnasbordas,p,0); }
    }

  for(int i=0; i<height; i++){
      if(semobjnasbordas.at<uchar>(i,width-1) == 255){ p.x=width-1; p.y=i; cv::floodFill(semobjnasbordas,p,0); }
    }
    
  for(int j=0; j<width; j++){
      if(semobjnasbordas.at<uchar>(0,j) == 255){ p.x=j; p.y=0; cv::floodFill(semobjnasbordas,p,0); }
    }

  for(int j=0; j<width; j++){
      if(semobjnasbordas.at<uchar>(height-1,j) == 255){ p.x=j; p.y=height-1; cv::floodFill(semobjnasbordas,p,0); }
    }

    image = semobjnasbordas.clone();

    // busca objetos presentes
  for(int i=0; i<height; i++){
    for(int j=0; j<width; j++){
      if(image.at<uchar>(i,j) == 255){
        // achou um objeto
        p.x=j; p.y=i; nobjects++;
  		  // preenche o objeto com o contador
		    cv::floodFill(image,p,nobjects);
      }
    }
  }

    //alterar a cor de fundo da imagem de preto para branco
    p.x=0; p.y=0; cv::floodFill(image,p,255);

    cv::imshow("image fundo branco", image);

    //Contagem de objetos com buracos
  for(int i=0; i<height; i++){
    for(int j=0; j<width; j++){
      if(image.at<uchar>(i,j) == 0){ 
        objcomburacos++;
        for(int k=j; k<width; k++){
          if(image.at<uchar>(i,k) != 0){ p.x=k; p.y=i; cv::floodFill(image,p,0); cv::floodFill(image,p,255); break;}
        }
      }
    }
  }

  cv::imshow("image buracos preenchidos", image);

  std::cout << "a figura tem " << nobjects << " bolhas, sendo " << objcomburacos << " bolhas com buracos\n";
  //cv::equalizeHist(image, realce);
  cv::imshow("imagem original clone", original);
  //cv::imshow("image", image);
  cv::imshow("sem objetos que tocam as bordas", semobjnasbordas);
  cv::imwrite("labeling.png", image);
  cv::waitKey();
  return 0;
}
```

### 3 - Manipulação de histogramas

#### Exercício 3.1 - equalize.cpp

Código [equalize.cpp]():

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

int main(int argc, char** argv){
  Mat image, escalacinza, escalacinzaeq, histCinza, histCinzaeq;
  int width, height;
  VideoCapture cap;
  int nbins = 64;
  float range[] = {0, 255};
  const float *histrange = { range };
  bool uniform = true;
  bool acummulate = false;
  int key;

	cap.open(0);
  if(!cap.isOpened()){ cout << "cameras indisponiveis"; return -1; }

  cap.set(CAP_PROP_FRAME_WIDTH, 640);
  cap.set(CAP_PROP_FRAME_HEIGHT, 480);
  width = cap.get(CAP_PROP_FRAME_WIDTH);
  height = cap.get(CAP_PROP_FRAME_HEIGHT);

  cout << "largura = " << width << endl;
  cout << "altura  = " << height << endl;

  int histw = nbins, histh = nbins/2;
  Mat histImgCinza(histh, histw, CV_8UC1, Scalar(0));
  Mat histImgCinzaEq(histh, histw, CV_8UC1, Scalar(0));

  while(1){
    cap >> image;
    cvtColor(image,escalacinza, COLOR_RGB2GRAY);

    //Equalização
    equalizeHist(escalacinza, escalacinzaeq);

    calcHist(&escalacinza, 1, 0, Mat(), histCinza, 1, &nbins, &histrange, uniform, acummulate);
    calcHist(&escalacinzaeq, 1, 0, Mat(), histCinzaeq, 1, &nbins, &histrange, uniform, acummulate);

    normalize(histCinza, histCinza, 0, histImgCinza.rows, NORM_MINMAX, -1, Mat());
    normalize(histCinzaeq, histCinzaeq, 0, histImgCinzaEq.rows, NORM_MINMAX, -1, Mat());

    histImgCinza.setTo(Scalar(0));
    histImgCinzaEq.setTo(Scalar(0));

    for(int i=0; i<nbins; i++){
      line(histImgCinza, Point(i, histh), Point(i, histh-cvRound(histCinza.at<float>(i))), Scalar(255, 255, 255), 1, 8, 0);
      line(histImgCinzaEq, Point(i, histh), Point(i, histh-cvRound(histCinzaeq.at<float>(i))), Scalar(255, 255, 255), 1, 8, 0);
    }

    histImgCinza.copyTo(escalacinza(Rect(0, 0, nbins, histh)));
    histImgCinzaEq.copyTo(escalacinzaeq(Rect(0, 0, nbins, histh)));

    imshow("image", escalacinza);
    imshow("imagem escala de cinza", escalacinzaeq);
    key = waitKey(30);
    if(key == 27) break;
    if(key == 32) { imwrite("Imagem em cinza.png", escalacinza); imwrite("Equalizada.png", escalacinzaeq);break;}
  }
  return 0;
}
```

#### Exercício 3.2 - motiondetector.cpp

### 4 - Filtragem no domínio espacial i

#### Exercício 4.1 - laplgauss.cpp

### 5 - Filtragem no domínio espacial ii

#### Exercício 5.1 - tiltshift.cpp

#### Exercício 5.2 - tiltshiftvideo.cpp

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/Albertotavaresdeoliveira/PDI---UFRN/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.

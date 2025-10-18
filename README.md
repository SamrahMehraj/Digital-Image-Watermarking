# Digital-Image-Watermarking
#Robust image Watermarking
clc; 
clear all; 
close all;
watermark = imread(logo_8x8.bmp);
t2=reshape(watermark, 1,64);

image=imread('lena.bmp');
image=imresize(image, [128 128]);
[rows cols dim]=size(image);
k=15;
    ycbcr=rgb2ycbcr(image);
    lum=ycbcr(:,:,1);
    cb=ycbcr(:,:,2);
    cr=ycbcr(:,:,3);
els = {'p',[-0.125 0.125],0};
lshaarInt = liftwave('haar','int2int');
lsnewInt = addlift(lshaarInt,els);
[A, B, C, D] = lwt2(double(lum),lsnewInt)

ylabel(['k ', num2str(k),' '])
 I=double(A(:,:,:));
[row col]=size(I);
blocksize1=16;
blocksize2=blocksize1/2;
nblocks=row*col/(blocksize2^2);

a=0; i=3;j=3;l=3;m=2;
p=0.2;th=60;sv=.9;%k=12;    %scaling variable

marked=[];
for x=1:blocksize1:row-(blocksize1-1)
    for y= 1:blocksize1:col-(blocksize1-1);
        block_bs1=I(x:x+blocksize1-1,y:y+blocksize1-1);
        block1=(block_bs1(1:blocksize2,1:blocksize2));
        block2=(block_bs1(1:blocksize2,blocksize2+1:2*blocksize2));
        block3=(block_bs1(blocksize2+1:2*blocksize2,1:blocksize2));
        block4=(block_bs1(blocksize2+1:2*blocksize2,blocksize2+1:2*blocksize2));
        
      for z=1:4
                                if z==1
                                   blocka=block1; blockb=block2;
                                elseif z==2
                                   blocka=block2; blockb=block4;
                                elseif z==3
                                   blocka=block4; blockb=block3;
                                else
                                   blocka=block3; blockb=block1;
                                end
   
  dc=blocka(1,1);
  zigzag13=[blocka(1,2) blocka(2,1) blocka(3,1) blocka(2,2) blocka(1,3) blocka(1,4) blocka(2,3) blocka(3,2) blocka(4,1) blocka(5,1) blocka(4,2) blocka(3,3) blocka(2,4)];
                              
                                      med=mode(zigzag13);
                                         Ppr=p;     %initial p=0.2
                                     if abs(med)>abs(dc)
                                           med=dc;
                                     end
                                     if abs(dc)>1000
                                        p=sv*med; %least
                                     elseif abs(dc)<1
                                        p=sv*med; %least
                                     else 
                                         p=sv*(dc-med)/dc; %most
                                     end
                                     if p<0.2
                                     p=Ppr; %few
                                     end
%% Embed_________________
                 a=a+1;
                                    if t2(a)==0        %for embedding 0

                                     if blocka(i,j)-blockb(l,m)>=th-k   %80-k
                                        while  blocka(i,j)-blockb(l,m)>=th-k
                                        blocka(i,j)=blocka(i,j)-p;
                                        blockb(l,m)=blockb(l,m)+p;

                                        end
       elseif  (blocka(i,j)-blockb(l,m)<=k) && ( blocka(i,j)-blockb(l,m)>=-th/2)
                       while   blocka(i,j)-blockb(l,m)<=k
                               blocka(i,j)=blocka(i,j)+p;
                               blockb(l,m)=blockb(l,m)-p;

                        end
          elseif  blocka(i,j)-blockb(l,m)<-th/2 
                     while  blocka(i,j)-blockb(l,m)>=-th-k
                                blocka(i,j)=blocka(i,j)-p;
                                 blockb(l,m)=blockb(l,m)+p;

                       end
               end

                                else           %w(x)=1  

                                   if blocka(i,j)-blockb(l,m)>=th/2
                                        while  ~(blocka(i,j)-blockb(l,m)>th+k)
                                                blocka(i,j)=blocka(i,j)+p;
                                                blockb(l,m)=blockb(l,m)-p;

                                        end
                  elseif  (blocka(i,j)-blockb(l,m)>=-k) &&  (blocka(i,j)-blockb(l,m)<th/2)
                               while  ~(blocka(i,j)-blockb(l,m)<-k)
                                        blocka(i,j)=blocka(i,j)-p;
                                        blockb(l,m)=blockb(l,m)+p;

                                    end
                                    elseif  blocka(i,j)-blockb(l,m)<=-th+k
                                         while  ~(blocka(i,j)-blockb(l,m)>-th+k)
                                                 blocka(i,j)=blocka(i,j)+p;
                                                 blockb(l,m)=blockb(l,m)-p;

                                            end
                                       end
                                    end
                                    
                                     if z==1
                                  block1=blocka; block2=blockb;
                                elseif z==2
                                  block2 =blocka; block4=blockb;
                                elseif z==3
                                   block4=blocka; block3=blockb;
                                else
                                   block3=blocka; block1=blockb;
                                end
      
        
      end
      
marked(x:x+blocksize1-1,y:y+blocksize1-1)=[(block1) (block2);(block3) (block4)];
    
    end
end
lum1=ilwt2(marked,B,C,D,lsnewInt);
 marked=uint8(lum1);
ycbcr_robust=cat(3,marked, cb,cr);
watermarked_image=ycbcr2rgb(ycbcr_robust);

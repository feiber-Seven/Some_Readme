.c -- --> .i -- --> .s -- --> .o -- --> .out

����ָ��: man xxx

echo $x:
    $# ��ʾ����������
    $0 �ǽű���������֡�
    $1 �Ǵ��ݸ���shell�ű��ĵ�һ��������
    $2 �Ǵ��ݸ���shell�ű��ĵڶ���������
    $@ ��ʾ���в������������в������Ƕ����ġ�
    $$ �ǽű����еĵ�ǰ����ID�š�
    $? ����ʾ���������˳�״̬��0��ʾû�д���������ʾ�д���

man: help && read manual

gcc --help:
    -o <����ļ�> ��ָ������ļ�
    -E ִֻ�б���Ԥ����
    -S ������ת��Ϊ���
    -c ֻ���б�����������������Ӳ���
    -Wall ��ʾ������Ϣ
    -l�������ĸ ����gcc��Ĭ�Ͽ⺯��

vim:

tmux:
    ![Alt text](20231017170324.png)
    ![Alt text](20231017170339.png)
    ![Alt text](20231017170412.png)

objdump:
    | less

git:
    git clone: 1.(-54) --> apt-get install libssl-dev
               2.(443) --> git config --global --unset http.proxy
    git branch: ��ʾ�������з�֧ (-m xxx): ��������ǰ��֧Ϊxxx
    git checkout -b pa0: �����µķ�֧pa0
    git status: �����־
    git diff: ����һ���ύ�Ĳ��챨��
    git add: �ύ���ݴ���
    git commit: �ݴ����ύ����
    git log: �鿴��ʷ��¼

bash shell
    PATH --> .bashrc

ccache: (which gcc)

make:
    (time) make (-j?): ����ʱ��Ͷ��ٸ�cpu

fzf: choose file
    vim $(fzf) vim��fzfѡ����ļ�

grep: g(all file) / re(����ʽ) / p(print)

find:
    find . -name "*.c" | xargs grep --color -nse '\<main\>' ��.C�ļ������ҵ�main ��\�ָ�����| xargs�ܵ�ǰһ�������Ϊ��һ�����룩

tree:
    ![Alt text](9M@2~13@MQ(%7DR)K%5D6%25XY3B8.png)
    ![Alt text](<LK8IW%L(`B(0_H%D}%KSBZV.png>)
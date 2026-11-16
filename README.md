Este projeto foi desenvolvido por alunos com o objetivo de criar uma aplicação Java capaz de se comunicar com a impressora Elgin i9, utilizando comandos ESC/POS, testes de impressão e interação via porta de comunicação.

Tecnologias Utilizadas
Java 17
IntelliJ IDEA
Biblioteca ESC/POS
Driver/SDK da Elgin i9

autores
charles victor
mauricio gomes
nicolas geraldo

import com.sun.jna.Library;
import com.sun.jna.Native;
import java.sql.SQLOutput;
import java.util.Scanner;
import java.nio.file.*;

public class Main {

    // Interface que representa a DLL, usando JNA
    public interface ImpressoraDLL extends Library {
        ImpressoraDLL INSTANCE = (ImpressoraDLL) Native.load(
                Paths.get("E1_Impressora01.dll").toAbsolutePath().toString(), ImpressoraDLL.class); // DLL na raiz do projeto

        int AbreConexaoImpressora(int tipo, String modelo, String conexao, int param);

        int FechaConexaoImpressora();

        int ImpressaoTexto(String dados, int posicao, int estilo, int tamanho);

        int Corte(int avanco);

        int ImpressaoQRCode(String dados, int tamanho, int nivelCorrecao);

        int ImpressaoCodigoBarras(int tipo, String dados, int altura, int largura, int HRI);

        int AvancaPapel(int linhas);

        int StatusImpressora(int param);

        int AbreGavetaElgin(int pino, int ti, int tf);

        int AbreGaveta(int pino, int ti, int tf);

        int SinalSonoro(int qtd, int tempoInicio, int tempoFim);

        int ModoPagina();

        int LimpaBufferModoPagina();

        int ImprimeModoPagina();

        int ModoPadrao();

        int PosicaoImpressaoHorizontal(int posicao);

        int PosicaoImpressaoVertical(int posicao);

        int ImprimeXMLSAT(String dados, int param);

        int ImprimeXMLCancelamentoSAT(String dados, String assQRCode, int param);
    }

    private static boolean conexaoAberta = false;
    private static int tipo;
    private static String modelo;
    private static String conexao;
    private static int parametro;
    private static final Scanner ler = new Scanner(System.in); // Scanner estático e final

    // Função para capturar entradas do usuário
    private static String capturarEntrada(String mensagem) {
        System.out.print(mensagem);
        return ler.nextLine();
    }

    // Função para configurar conexão
    public static void configurarConexao() {
        System.out.println("digite o tipo da impressora (numero)");
        tipo = ler.nextInt();
        ler.nextLine();
        System.out.println("digite modelo");
        modelo = ler.nextLine();
        System.out.println("digite a porta");
        conexao = ler.nextLine();
        System.out.println("digite o parametro (numero)");
        parametro = ler.nextInt();
        System.out.println("dados armazenados");
    }

    public static void abrirConexao() {
        // Função para abrir a conexão com a impressora
        if (!conexaoAberta) {
            int retorno = ImpressoraDLL.INSTANCE.AbreConexaoImpressora(tipo, modelo, conexao, parametro);
            if (retorno == 0) {
                conexaoAberta = true;
                System.out.println("Conexão aberta com sucesso.");
            } else {
                System.out.println("Erro ao abrir conexão. Código de erro: " + retorno);
            }
        } else {
            System.out.println("Conexão já está aberta.");
        }
    }

    public static void fecharConexao() {
        if (conexaoAberta == true) {
            conexaoAberta = false;
        }
    }



    public static void exibirMenu() {
        System.out.println("\n*************************************************");
        System.out.println("**************** MENU IMPRESSORA *******************");
        System.out.println("*************************************************\n");

        System.out.println("1  - Configurar Conexao");
        System.out.println("2  - abrir conexão");
        System.out.println("3  - Imprimir texto");
        System.out.println("4  - Imprimir QRcode");
        System.out.println("5  - Imprimir codigo de barras");
        System.out.println("6  - Imprimir cupom");
        System.out.println("7  - Cancelar cupom");
        System.out.println("8  - Abrir gaveta");
        System.out.println("9  - Sinal sonoro");
        System.out.println("10  - Avançar papel");
        System.out.println("0  - Encerrar programa");

    }
    public static String login() {
        System.out.println("digite usuario e senha");
        for (int i = 3; i >= 0; i--) {
            if (i == 0) {
                System.out.println("acabou suas tentativas \nfinalizando sistema");
                System.exit(0);
            }
            if (i < 3) {
                System.out.println("Senha incorreta, tente novamente");
            }
            System.out.println("voce tem " + i + " tentativas");
            System.out.println("digite seu usuario:");
            String usuario = ler.nextLine();
            System.out.println("digite sua senha: ");
            String senha = ler.nextLine();
            if (usuario.equals("admin") && senha.equals("1234")) {
                break;
            }
        }
        return "sucesso";
    }

    public static void imprimirtexto(){
        if (!conexaoAberta) {
            System.out.println("Conexão fechada. Abra a conexão antes (opção 2).");
            return;
        }
        String texto = capturarEntrada("Texto a imprimir: ");
        int copias = lerInteiro("quantidade de copias: ");
        for (int i = 1; i <= copias; i++) {
            int retorno = ImpressoraDLL.INSTANCE.ImpressaoTexto(texto, 1, 4, 0);
            if (retorno >= 0) {
                System.out.println("Cópia " + i + " impressa!");
            } else {
                System.out.println("Erro ao imprimir. Código: " + retorno);
            }
        }
        ImpressoraDLL.INSTANCE.Corte(5);
    }

    public static void imprimircodigodebarras(){
        if (conexaoAberta == true){
            int retorno = ImpressoraDLL.INSTANCE.ImpressaoCodigoBarras(8, "{A12345678912", 100, 2, 3);
            if (retorno == 0){
                ImpressoraDLL.INSTANCE.Corte(3);
                System.out.println("codigo Barras impresso com sucesso");
            }else{
                System.out.println("Erro ao imprimir codigo de barras. Retorno " + retorno);
            }
        }else{
            System.out.println("não existe conexão em aberto");
        }

    }
    public static void imprimirQRCode(){
        if (conexaoAberta == true){
            String texto = capturarEntrada("Texto a imprimir: ");
            int retorno = ImpressoraDLL.INSTANCE.ImpressaoQRCode(texto, 6, 4);
            if (retorno == 0){
                ImpressoraDLL.INSTANCE.Corte(5);
                System.out.println("QRcode impresso com sucesso");
            }else{
                System.out.println("Erro ao imprimir QRcode. Retorno " + retorno);
            }
        }else{
            System.out.println("não existe conexão em aberto");
        }

    }
    public static void imprimirXMLSAT(){
        if (conexaoAberta == true){
            String cup =
            int retorno = ImpressoraDLL.INSTANCE.ImprimeXMLSAT("xml", 0);
            if (retorno == 0){
                ImpressoraDLL.INSTANCE.Corte(5);
                System.out.println("cupom impresso com sucesso");
            }else{
                System.out.println("Erro ao imprimir cupom. Retorno " + retorno);
            }
        }else{
            System.out.println("não existe conexão em aberto");
        }

    }
    public static void imprimirXMLCancelamentoSAT(){
        if (conexaoAberta == true){
            int retorno = ImpressoraDLL.INSTANCE.ImprimeXMLCancelamentoSAT("xml", "12345", 0);
            if (retorno == 0){
                ImpressoraDLL.INSTANCE.Corte(5);
                System.out.println("cupom cancelado com sucesso");
            }else{
                System.out.println("Erro ao cancelar cupom. Retorno " + retorno);
            }
        }else{
            System.out.println("não existe conexão em aberto");
        }

    }
    public static void abrirGaveta(){
        if (conexaoAberta == true){
            int retorno = ImpressoraDLL.INSTANCE.AbreGavetaElgin(1, 5, 10);
            if (retorno == 0){
                ImpressoraDLL.INSTANCE.Corte(5);
                System.out.println("gaveta aberta com sucesso");
            }else{
                System.out.println("Erro ao abrir gaveta. Retorno " + retorno);
            }
        }else{
            System.out.println("não existe conexão em aberto");
        }

    }
    public static void sinalSonoro() {
        if (!conexaoAberta) {
            System.out.println("Conexão fechada. Abra a conexão antes (opção 2).");
            return;
        }
        int quantidade = lerInteiro("Quantos bips? ");
        int retorno = ImpressoraDLL.INSTANCE.SinalSonoro(quantidade, 50, 5);
        if (retorno == 0) {
            System.out.println("Bip!");
        } else {
            System.out.println("Erro no sinal sonoro. Código: " + retorno);
        }
    }

    public static void avancarpapel(){
        if (conexaoAberta == true){
            int avanca = lerInteiro("quantas linhas? ");
            int retorno = ImpressoraDLL.INSTANCE.AvancaPapel(avanca);
            if (retorno == 0){
                ImpressoraDLL.INSTANCE.Corte(avanca);
                System.out.println("avançou " + avanca + " linhas");
            }else{
                System.out.println("Erro ao avançar papel. Retorno " + retorno);
            }
        }else{
            System.out.println("não existe conexão em aberto");
        }

    }

    public static int lerInteiro(String lerint){
        System.out.print(lerint);

        while (!ler.hasNextInt()) {
            System.out.println("Por favor, digite apenas números: ");
            ler.next();
        }
        return ler.nextInt();
    }




    public static void main(String[] args) throws java.io.IOException {
        login();
        System.out.println("login feito com sucesso");
        while (true) {
            exibirMenu();
            String escolha = capturarEntrada("\nDigite a opção desejada: ");

            if (escolha.equals("0")) {
                fecharConexao();
                System.out.println("Programa encerrado.");
                break;
            } else if (escolha.equals("1")) {
                System.out.println("você escolheu configurar sistema");
                configurarConexao();
            } else if (escolha.equals("2")) {
                System.out.println("você escolheu abrir conexão");
                abrirConexao();

            } else if (escolha.equals("3")) {
                System.out.println("você escolheu imprimir texto");
                imprimirtexto();

            } else if (escolha.equals("4")) {
                System.out.println("você escolheu imprimir QRcode");
                imprimirQRCode();

            } else if (escolha.equals("5")) {
                System.out.println("você escolheu imprimir codigo de barras");
                imprimircodigodebarras();

            } else if (escolha.equals("6")) {
                System.out.println("você escolheu imprimir cupom");
                imprimirXMLSAT();

            } else if (escolha.equals("7")) {
                System.out.println("você escolheu cancelar cupom");
                imprimirXMLCancelamentoSAT();

            } else if (escolha.equals("8")) {
                System.out.println("você escolheu abrir gaveta");
                abrirGaveta();

            } else if (escolha.equals("9")) {
                System.out.println("você escolheu sinal sonoro");
                sinalSonoro();

            } else if (escolha.equals("10")) {
                System.out.println("você escolheu avançar papel");
                avancarpapel();

            } else {
                System.out.println("opção invalida ERRO");
            }
        }

        ler.close(); // Fecha o scanner ao final do programa
    }
}


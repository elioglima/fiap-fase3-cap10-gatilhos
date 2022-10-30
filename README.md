# ATIVIDADE EM GRUPO

## Cap 10 - Gatilhos mágicos - Exercendo a Automação

# FUNCIONALIDADES APRESENTADAS

- MASCARA DE CEP,
- MASCARA DE RG,
- MASCARA DE CPF,
- MASCARA DE CNPJ,
- MASCARA DE TELEFONE,
- MASCARA DE CELULAR,

* FUNCAO PARA CALCULAR A IDADE
* FUNCAO PARA FORMATAR CPF
* FUNCAO PARA FORMATAR CNPJ

* TRIGGER PARA REGISTRO DE LOGS NA HORA DO ADICIONAR UM CADASTRO DE PESSOA FISICA
* TRIGGER PARA REGISTRO DE LOGS NA HORA DO REMOVER UM CADASTRO DE PESSOA FISICA
* TRIGGER PARA REGISTRO DE LOGS NA HORA DO ALTERAR UM CADASTRO DE PESSOA FISICA

# PREPARACAO DO BANCO

       DROP TABLE IF EXISTS PESSOAFISICA;
       DROP TABLE IF EXISTS LOGS;

       DROP TRIGGER IF EXISTS TRG_PESSOAFISICA_LOGS_ADICIONA;

       DROP FUNCTION IF EXISTS MASCARA;
       DROP FUNCTION IF EXISTS FUN_CALCULAIDADE;
       DROP FUNCTION IF EXISTS FORMATAR_CPF;
       DROP FUNCTION IF EXISTS FORMATAR_CNPJ;

# TABELA DE CADASTRO DE PESSOA FISICA

    CREATE TABLE PESSOAFISICA (
           ID NUMBER(*, 0) GENERATED ALWAYS AS IDENTITY INCREMENT BY 1 MAXVALUE 999999 MINVALUE 1 CACHE 20 NOT NULL,
           NOME VARCHAR2(100 BYTE) NOT NULL,
           CPF VARCHAR2(20 BYTE) NOT NULL,
           RG VARCHAR2(20 BYTE) NOT NULL,
           TELEFONE VARCHAR2(15 BYTE) NOT NULL,
           CELULAR VARCHAR2(15 BYTE) NOT NULL,
           DATA_NASCIMENTO DATE NOT NULL,
           CONSTRAINT PESSOAFISICA_PK PRIMARY KEY (ID) USING INDEX (
                  CREATE UNIQUE INDEX PESSOAFISICA_PK ON PESSOAFISICA (ID ASC) LOGGING TABLESPACE TBSPC_ALUNOS PCTFREE 10 INITRANS 2 STORAGE (
                         INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS UNLIMITED BUFFER_POOL DEFAULT
                  ) NOPARALLEL
           ) ENABLE
    ) LOGGING TABLESPACE TBSPC_ALUNOS PCTFREE 10 INITRANS 1 STORAGE (
           INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS UNLIMITED BUFFER_POOL DEFAULT
    ) NOCOMPRESS NO INMEMORY NOPARALLEL;

# TABELA DE LOGS

    CREATE TABLE LOGS (
           ID NUMBER(*, 0) GENERATED ALWAYS AS IDENTITY INCREMENT BY 1 MAXVALUE 999999 MINVALUE 1 CACHE 20 NOT NULL,
           DESCRICAO VARCHAR2(1000 BYTE) NOT NULL,
           FUNCIONALIDADE VARCHAR2(45 BYTE) NOT NULL,
           DATA TIMESTAMP(6) DEFAULT CURRENT_TIMESTAMP NOT NULL,
           ID_REFERENCIA NUMBER(*, 0)
    ) LOGGING TABLESPACE TBSPC_ALUNOS PCTFREE 10 INITRANS 1 STORAGE (
           INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS UNLIMITED BUFFER_POOL DEFAULT
    ) NOCOMPRESS NO INMEMORY NOPARALLEL;

# TRIGGER DE REGISTRO DE LOGS ADICIONAR

    CREATE
    OR REPLACE TRIGGER TRG_PESSOAFISICA_LOGS_ADICIONA AFTER INSERT ON PESSOAFISICA FOR EACH ROW BEGIN
    INSERT INTO
           LOGS (FUNCIONALIDADE, ID_REFERENCIA, DESCRICAO, DATA)
    VALUES
           (
                  'NOVO CADASTRO DE PESSOA FISICA',
    : new.ID,
    : new.NOME,
                  CURRENT_TIMESTAMP
           );

    END;

# TRIGGER DE REGISTRO DE LOGS REMOVE

    CREATE
    OR REPLACE TRIGGER TRG_PESSOAFISICA_LOGS_REMOVE AFTER DELETE ON PESSOAFISICA FOR EACH ROW BEGIN
    INSERT INTO
           LOGS (FUNCIONALIDADE, ID_REFERENCIA, DESCRICAO, DATA)
    VALUES
           (
                  'REMOVE CADASTRO DE PESSOA FISICA',
    : old.ID,
    : old.NOME,
                  CURRENT_TIMESTAMP
           );

    END;

# TRIGGER DE REGISTRO DE LOGS ATUALIZA

    CREATE
    OR REPLACE TRIGGER TRG_PESSOAFISICA_LOGS_ATUALIZA AFTER
    update
           ON PESSOAFISICA FOR EACH ROW BEGIN
    INSERT INTO
           LOGS (FUNCIONALIDADE, ID_REFERENCIA, DESCRICAO, DATA)
    VALUES
           (
                  'NOVO CADASTRO DE PESSOA FISICA',
    : new.ID,
    : new.NOME,
                  CURRENT_TIMESTAMP
           );

    END;

# FUNCAO GENERICA DE MASCARA

- MASK("12345123", "#####-###") AS cep,
- MASK("121231231", "##.###.###-#") AS rg,
- MASK("1212312312", "##.###.###-##") AS cpf,
- MASK("12123123123412", "##.###.###/####-##") AS cnpj,
- MASK("1212341234", "(##) ####-####") AS telefone,
- MASK("12123451234", "(##) #####-####") AS celular

# FUNCAO GENERICA

    CREATE OR REPLACE FUNCTION MASCARA
        (val IN VARCHAR2, mask IN VARCHAR2) RETURN VARCHAR2 AS maskared VARCHAR(100) DEFAULT ' ';

    k INT DEFAULT 0;
    i INT DEFAULT 0;

    BEGIN WHILE i < LENGTH(mask) LOOP i: = i + 1;
        IF SUBSTR(mask, i, 1) = '#' THEN
          IF k < LENGTH(val) THEN k: = k + 1;
            maskared: = CONCAT(maskared, SUBSTR(val, k, 1));
          END IF;

        ELSE
          IF i < LENGTH(mask) THEN
            maskared: = CONCAT(maskared, SUBSTR(mask, i, 1));
          END IF;

        END IF;
      END LOOP;

      RETURN maskared;

    END MASCARA;

# RETORNA A IDADE

    CREATE
    OR REPLACE FUNCTION FUN_CALCULAIDADE (DATA_NASCIMENTO IN DATE) RETURN NUMBER AS BEGIN RETURN trunc(
           (months_between(CURRENT_DATE, DATA_NASCIMENTO)) / 12
    );

    END FUN_CALCULAIDADE;

# OUTRO MODELO FORMATACAO CPF

    CREATE OR REPLACE FUNCTION FORMATAR_CPF (CPF IN VARCHAR2)

        RETURN VARCHAR2 AS BEGIN RETURN regexp_replace(
           LPAD(CPF, 11),
           '([0-9]{3})([0-9]{3})([0-9]{3})',
           '\1.\2.\3-'
        );

    END FORMATAR_CPF;

# OUTRO MODELO FORMATACAO CNPJ

    CREATE OR REPLACE FUNCTION FORMATAR_CNPJ (CNPJ IN VARCHAR2)

        RETURN VARCHAR2 AS BEGIN RETURN regexp_replace(
           LPAD(CNPJ, 14),
           '([0-9]{2})([0-9]{3})([0-9]{3})([0-9]{4})',
           '\1.\2.\3/\4-'
      );

    END FORMATAR_CNPJ;

COMMIT;

# INSERINDO DADOS

    INSERT INTO
           pessoafisica (nome, cpf, rg, telefone, celular, data_nascimento)
    values
           (
                  'Joao Almenda Brandao',
                  '64443404031',
                  '451089923',
                  '1120823541',
                  '11952550331',
                  TO_DATE('1979-11-04', 'YYYY-MM-DD')
           );

    INSERT INTO
           pessoafisica (nome, cpf, rg, telefone, celular, data_nascimento)
    values
           (
                  'Elisa Gomes da Silva',
                  '98771181067',
                  '228949609',
                  '1120823541',
                  '11952550331',
                  TO_DATE('1986-03-11', 'YYYY-MM-DD')
           );

    INSERT INTO
           pessoafisica (nome, cpf, rg, telefone, celular, data_nascimento)
    values
           (
                  'Pedro Alencar dos Santos',
                  '05955141030',
                  '358771663',
                  '1120823541',
                  '11952550331',
                  TO_DATE('1955-09-07', 'YYYY-MM-DD')
           );

    INSERT INTO
           pessoafisica (nome, cpf, rg, telefone, celular, data_nascimento)
    values
           (
                  'Abigail da Costa',
                  '78766609089',
                  '355373415',
                  '1120823541',
                  '11952550331',
                  TO_DATE('1965-02-01', 'YYYY-MM-DD')
           );

# CONSULTAS

# VISUALIZA OS LOGS

    SELECT
           l.*
    FROM
           logs l;

# LISTA PESSOA FISICA

    SELECT
           PF.*,
           FUN_CALCULAIDADE(PF.DATA_NASCIMENTO) AS IDADE
    FROM
           PESSOAFISICA PF;

    SELECT
              FUN_CALCULAIDADE(TO_DATE('1979-11-04', 'YYYY-MM-DD')) AS IDADE,
              MASCARA('12345123', '#####-###') AS cep,
              MASCARA('121231231', '##.###.###-#') AS rg,
              MASCARA('1212312312', '##.###.###-##') AS cpf,
              MASCARA('12123123123412', '##.###.###/####-##') AS cnpj,
              MASCARA('1212341234', '(##) ####-####') AS telefone,
              MASCARA('12123451234', '(##) #####-####') AS celular
       FROM dual;

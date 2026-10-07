# banco-de-dados-

use MEUBANCO;
CREATE TABLE endereco (
    id INT PRIMARY KEY AUTO_INCREMENT,
    rua VARCHAR(100) NOT NULL,
    numero VARCHAR(10),
    bairro VARCHAR(50),
    cidade VARCHAR(50),
    estado VARCHAR(2),
    cep VARCHAR(10)
);

-- 2. Cliente
CREATE TABLE cliente (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    telefone VARCHAR(20),
    endereco_id INT UNIQUE,
    FOREIGN KEY (endereco_id) REFERENCES endereco(id) ON DELETE SET NULL
);

-- 3. Fornecedor
CREATE TABLE fornecedor (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome_empresa VARCHAR(100) NOT NULL,
    telefone VARCHAR(20)
);

-- 4. Produto
CREATE TABLE produto (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10, 2) NOT NULL,
    quantidade_estoque INT NOT NULL DEFAULT 0,
    fornecedor_id INT,
    FOREIGN KEY (fornecedor_id) REFERENCES fornecedor(id)
);

-- 5. Pedido (Contém tipo de entrega e status)
CREATE TABLE pedido (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT NOT NULL,
    tipo_pedido ENUM('ENTREGA', 'RETIRADA') NOT NULL DEFAULT 'ENTREGA',
    status ENUM('PENDENTE', 'EM_PREPARACAO', 'PRONTO_PARA_RETIRADA', 'SAIU_PARA_ENTREGA', 'CONCLUIDO', 'CANCELADO') DEFAULT 'PENDENTE',
    valor_total DECIMAL(10, 2) DEFAULT 0.00,
    data_pedido DATETIME DEFAULT CURRENT_TIMESTAMP,
    data_conclusao DATETIME,
    FOREIGN KEY (cliente_id) REFERENCES cliente(id)
);

-- 6. Itens do Pedido
CREATE TABLE item_pedido (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedido_id INT NOT NULL,
    produto_id INT NOT NULL,
    quantidade INT NOT NULL CHECK (quantidade > 0),
    preco_unitario DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (pedido_id) REFERENCES pedido(id) ON DELETE CASCADE,
    FOREIGN KEY (produto_id) REFERENCES produto(id)
);

-- 7. Entregador
CREATE TABLE entregador (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    telefone VARCHAR(20),
    veiculo VARCHAR(30)
);

-- 8. Entrega (Usada apenas se tipo_pedido = 'ENTREGA')
CREATE TABLE entrega (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedido_id INT UNIQUE NOT NULL,
    entregador_id INT,
    data_saida DATETIME,
    data_entrega DATETIME,
    status_entrega ENUM('AGUARDANDO_RETIRADA', 'EM_TRANSITO', 'ENTREGUE', 'FALHOU') DEFAULT 'AGUARDANDO_RETIRADA',
    observacoes VARCHAR(255),
    FOREIGN KEY (pedido_id) REFERENCES pedido(id),
    FOREIGN KEY (entregador_id) REFERENCES entregador(id)
);

-- 9. Histórico de Mudanças de Status do Pedido (Rastreabilidade)
CREATE TABLE historico_status_pedido (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedido_id INT NOT NULL,
    status_anterior VARCHAR(30),
    status_novo VARCHAR(30) NOT NULL,
    data_mudanca DATETIME DEFAULT CURRENT_TIMESTAMP,
    observacao VARCHAR(255),
    FOREIGN KEY (pedido_id) REFERENCES pedido(id) ON DELETE CASCADE
);

-- 10. Histórico de Movimentação de Estoque
CREATE TABLE movimentacao_estoque (
    id INT PRIMARY KEY AUTO_INCREMENT,
    produto_id INT NOT NULL,
    tipo_movimentacao ENUM('ENTRADA', 'SAIDA') NOT NULL,
    quantidade INT NOT NULL,
    motivo VARCHAR(100) NOT NULL, -- Ex: "Venda - Pedido #123", "Reposição Fornecedor"
    data_movimentacao DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (produto_id) REFERENCES produto(id)
);

DELIMITER //

CREATE TRIGGER AtualizarEstoque
AFTER INSERT ON Itens_Pedido
FOR EACH ROW
BEGIN
    UPDATE Produtos
    SET quantidade_estoque = quantidade_estoque - NEW.quantidade
    WHERE produto_id = NEW.produto_id;
END //

DELIMITER ;

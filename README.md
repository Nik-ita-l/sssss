class Piece:
    """Базовый класс для всех  фигур"""
    def __init__(self, color, symbol):
        self.color = color
        self.symbol = symbol

    def __str__(self):
        return self.symbol

    def valid_moves(self, board, position):
        raise NotImplementedError("Subclasses should implement this method")


class Checker(Piece):
    """Класс шашки"""
    def __init__(self, color, symbol, is_king=False):
        super().__init__(color, symbol)
        self.is_king = is_king  #  является ли шашка дамкой

    def valid_moves(self, board, position):
        """Возвращает список допустимых ходов для шашки"""
        row, col = position # Текущая позиция шашки
        moves = []
        capture_moves = [] # Список ходов со взятием фигуры противника
        direction = -1 if self.color == 'white' else 1  # Направление движения: белые вверх, черные вниз

        # Проверка ходов для обычной шашки
        if not self.is_king:
            # Обычные ходы (без взятия)
            for dc in (-1, 1):  # Проверяем обе диагонали вперед
                r, c = row + direction, col + dc
                if 0 <= r < 8 and 0 <= c < 8 and not board[r][c]: # если клетка в пределах доски и она свободна то ход добовляется в moves
                    moves.append((r, c))

            # Ходы со взятием
            for dc in (-2, 2):  # Проверяем возможные прыжки через фигуру
                r, c = row + 2 * direction, col + dc # конечная клетка должна быть пустая
                mid_r, mid_c = row + direction, col + dc // 2
                if (0 <= r < 8 and 0 <= c < 8 and not board[r][c] and 
                    board[mid_r][mid_c] and board[mid_r][mid_c].color != self. #Клетка посередине (mid_r, mid_c) должна содержать фигуру противника (board[mid_r]
                    color):
                    capture_moves.append((r, c))
        else:
            # для дамки (может ходить в любом направлении)
            for dr in (-1, 1):
                for dc in (-1, 1):
                    r, c = row + dr, col + dc
                    while 0 <= r < 8 and 0 <= c < 8:
                        if not board[r][c]:
                            moves.append((r, c))
                        else:
                            if board[r][c].color != self.color and 0 <= r + dr < 8 and 0 <= c + dc < 8 and not board[r + dr][c + dc]:
                                capture_moves.append((r + dr, c + dc))
                            break
                        r += dr
                        c += dc

        # Если есть ходы со взятием, возвращаем только их (по правилам шашек)
        return capture_moves if capture_moves else moves


class CheckerKing(Checker):
    """Класс дамки в шашках (наследуется от обычной шашки)"""
    def __init__(self, color, symbol):
        super().__init__(color, symbol, is_king=True)


class Pawn(Piece):
    """Класс пешки"""
    def valid_moves(self, board, position):
        row, col = position
        moves = []
        direction = -1 if self.color == 'white' else 1 #направление движения пешeк, белая- вверх, черная- вниз
        start_row = 6 if self.color == 'white' else 1

        # Ход вперед
        if 0 <= row + direction < 8 and not board[row + direction][col]:
            moves.append((row + direction, col))
            if row == start_row and not board[row + 2 * direction][col]:
                moves.append((row + 2 * direction, col))

        # Взятие фигур
        for dc in (-1, 1): # смещение по столбцу (-1 — влево, 1 — вправо)
            r, c = row + direction, col + dc #координаты клетки по диагонали от пешки
            if 0 <= c < 8 and 0 <= r < 8:
                target = board[r][c]
                if target and target.color != self.color: #проверяем, что фигура вражеская
                    moves.append((r, c))

        return moves


class Rook(Piece):
    """Класс ладьи"""
    def valid_moves(self, board, position):
        row, col = position
        moves = []
        directions = ((-1, 0), (1, 0), (0, -1), (0, 1)) #кортеж из четырёх возможных направлений движения ладьи

        for dr, dc in directions: #смещение по строке (dr) и столбцу (dc) для текущего направления.
            r, c = row + dr, col + dc
            while 0 <= r < 8 and 0 <= c < 8: #Цикл выполняется, пока новые координаты (r, c) находятся в пределах доски
                target = board[r][c]
                if not target:
                    moves.append((r, c))
                else:
                    if target.color != self.color:
                        moves.append((r, c))
                    break
                r += dr
                c += dc

        return moves


class Knight(Piece):
    """Класс коня"""
    def valid_moves(self, board, position):
        row, col = position
        moves = []
        moves_pattern = ((-2, -1), (-1, -2), (1, -2), (2, -1),
                         (2, 1), (1, 2), (-1, 2), (-2, 1))

        for dr, dc in moves_pattern:
            r, c = row + dr, col + dc
            if 0 <= r < 8 and 0 <= c < 8:
                target = board[r][c]
                if not target or target.color != self.color: #Если клетка пустая (not target) или содержит фигуру противника (target.color != self.color), добавляем ход в список
                    moves.append((r, c))

        return moves


class Bishop(Piece):
    """Класс слона"""
    def valid_moves(self, board, position):
        row, col = position
        moves = []
        directions = ((-1, -1), (-1, 1), (1, -1), (1, 1))

        for dr, dc in directions:
            r, c = row + dr, col + dc
            while 0 <= r < 8 and 0 <= c < 8:
                target = board[r][c]
                if not target:
                    moves.append((r, c))
                else: #Если клетка занята:Если фигура противника - добавляем ход (можно бить) Прерываем цикл (нельзя прыгать через фигуры)
                    if target.color != self.color:
                        moves.append((r, c))
                    break
                r += dr
                c += dc

        return moves


class Queen(Piece):
    """Класс ферзя"""
    def valid_moves(self, board, position):
        rook = Rook(self.color, 'Q') #Создаётся временный объект ладьи того же цвета, что и ферзь
        bishop = Bishop(self.color, 'Q') #Аналогично создаётся временный объект слона того же цвета
        return rook.valid_moves(board, position) + bishop.valid_moves(board, position) # обьединяемая  ходы ладьи и слона и с помощью плюса делаем их едиными


class King(Piece):
    """Класс короля"""
    def valid_moves(self, board, position):
        row, col = position
        moves = []

        for dr in (-1, 0, 1):
            for dc in (-1, 0, 1):
                if dr == 0 and dc == 0: #Исключаем вариант, когда король остаётся на месте (0,0)
                    continue
                r, c = row + dr, col + dc
                if 0 <= r < 8 and 0 <= c < 8:
                    target = board[r][c] # Проверяем содержимое клетки. Получаем объект фигуры в целевой клетке (или None, если клетка пуста)
                    if not target or target.color != self.color:
                        moves.append((r, c))

        return moves


class Board:
    """Класс, представляющий шахматную доску и игровую логику"""
    def __init__(self, game_type='chess'):
        self.board = [[None for _ in range(8)] for _ in range(8)]
        self.move_history = [] #Создаётся пустой список для хранения истории ходов
        self.game_type = game_type  # 'chess' или 'checkers'
        self.setup_board() # метод который заполняет жоску фигурами на начальной позиции

    def setup_board(self):
        """Расставляет фигуры в начальную позицию"""
        if self.game_type == 'chess':
            # Черные фигуры
            self.board[0] = [
                Rook('black', 'r'), Knight('black', 'n'), Bishop('black', 'b'), Queen('black', 'q'),
                King('black', 'k'), Bishop('black', 'b'), Knight('black', 'n'), Rook('black', 'r')
            ]
            self.board[1] = [Pawn('black', 'p') for _ in range(8)] #Первая строка: 8 чёрных пешек
            
            # Белые фигуры
            self.board[6] = [Pawn('white', 'P') for _ in range(8)]
            self.board[7] = [
                Rook('white', 'R'), Knight('white', 'N'), Bishop('white', 'B'), Queen('white', 'Q'),
                King('white', 'K'), Bishop('white', 'B'), Knight('white', 'N'), Rook('white', 'R')
            ]
        elif self.game_type == 'checkers':
            # Расстановка шашек (классический вариант)
            for row in range(8):
                for col in range(8):
                    if (row + col) % 2 == 1:  # Шашки ставятся только на темные клетки
                        if row < 3:
                            self.board[row][col] = Checker('black', 'b')
                        elif row > 4:
                            self.board[row][col] = Checker('white', 'w')

    def display(self):
        """Отображает текущее состояние доски в терминале"""
        print("  a b c d e f g h")
        for i, row in enumerate(self.board): #Перебор строк доски enumerate(self.board) возвращает пары (индекс, строка) i - текущий индекс строки (0-7, где 0 - верх доски)row - список фигур в текущей строке
            print(f"{8 - i} ", end="") #8-i = номер ряда, end="" предотвращает перенос строки
            print(' '.join(str(piece) if piece else '.' for piece in row), f"{8 - i}") # Генератор проходит по всем элементам строки (piece in row)Если клетка содержит фигуру (piece не None), выводится её символ (str(piece))Если клетка пустая (piece is None), выводится точка '.'' '.join() объединяет элементы через пробел
        print("  a b c d e f g h")

    def move_piece(self, start, end, color):
        #Распаковываем координаты начальной и конечной позиций
        start_row, start_col = start
        end_row, end_col = end
        #Достаём объект фигуры из начальной клетки доски
        piece = self.board[start_row][start_col]

        if not piece or piece.color != color: #not piece - клетка пустая (нет фигуры) piece.color != color - фигура принадлежит противнику
            print("Некорректный ход: вы пытаетесь походить чужой фигурой или пустой клеткой.")
            return False

        valid_moves = piece.valid_moves(self.board, start)
        if (end_row, end_col) in valid_moves:
            self.move_history.append((start, end, self.board[end_row][end_col])) # сохраняем историю кодов
            
            # Перемещение фигуры
            self.board[end_row][end_col] = piece
            self.board[start_row][start_col] = None
            
            # Логика для шашек
            if self.game_type == 'checkers':
                # Проверка на превращение в дамку
                if isinstance(piece, Checker) and not piece.is_king:
                    if (piece.color == 'white' and end_row == 0) or (piece.color == 'black' and end_row == 7):
                        self.board[end_row][end_col] = CheckerKing(piece.color, piece.symbol.upper())
                
                # Удаление сбитой шашки (при прыжке через неё)
                if abs(start_row - end_row) == 2:  # Это был прыжок через шашку
                    mid_row = (start_row + end_row) // 2
                    mid_col = (start_col + end_col) // 2
                    self.board[mid_row][mid_col] = None
            
            return True
        
        print("Некорректный ход: фигура не может так ходить.")
        return False

    def undo_move(self):
        # Проверка наличия ходов для отмены
        if not self.move_history:
            print("Нет ходов для отмены.")
            return False
        
        # Извлечение последнего хода из истории
        start, end, captured = self.move_history.pop()
        
        # Возврат фигуры на начальную позицию
        self.board[start[0]][start[1]] = self.board[end[0]][end[1]]
        
        # Восстановление взятой фигуры (если была)
        self.board[end[0]][end[1]] = captured
        
        # Подтверждение успешной отмены
        return True

    def show_hints(self, position):
        #подсказки 
        #Получение позиции и фигуры
        row, col = position
        piece = self.board[row][col]
        current_player = 'white' if len(self.move_history) % 2 == 0 else 'black' #Определение текущего игрока
        
        if not piece or piece.color != current_player:
            print("Некорректная позиция для подсказки.")
            return
        
        moves = piece.valid_moves(self.board, (row, col)) #Вызывает метод valid_moves() для выбранной фигуры. Возвращает список возможных ходов в формате [(r1,c1), (r2,c2), ...]

        for r in range(8): # Перебор строк (0-7)
            for c in range(8): # Перебор столбцов (0-7)
                print('*' if (r, c) in moves else self.board[r][c] or '.', end=' ') # Перенос строки после каждого ряда
            print()


def main():
    # Выбор игры
    game_type = input("Здравсвуйте! Выберите игру в которую вы хотите поиграть (chess/checkers): ").strip().lower()
    while game_type not in ['chess', 'checkers']:
        print("Некорректный ввод. Пожалуйста проверьте правильностьность написания и введите 'chess' или 'checkers'.")
        game_type = input("Выберите игру (chess/checkers): ").strip().lower()
    
    # создание доски
    board = Board(game_type)
    
    # начинаем игру с белых фигур
    turn = 'white'
    
    # Основной игровой цикл
    while True:
        # Отображаем текущее состояние доски
        board.display()
        
        # Показываем, чей сейчас ход
        print(f"Ход {'белых' if turn == 'white' else 'черных'}")
        
        # Получаем ввод пользователя и обрабатываем его
        command = input("Введите команду (например, 'e2 e4', 'undo', 'hint e2'): ").strip().split()
        
        # Если ввод пустой - запрашиваем снова
        if not command:
            continue

        # Обработка команды отмены хода (undo)
        if command[0] == 'undo':
            # Если отмена прошла успешно, меняем игрока
            if board.undo_move():
                turn = 'black' if turn == 'white' else 'white'
            continue
            
        # Обработка команды подсказки (hint)
        if command[0] == 'hint':
            # Проверяем корректность формата команды
            if len(command) < 2:
                print("Некорректный ввод. Попробуйте снова.")
                continue
            try:
                # Преобразуем буквенную координату в числовую (a-h -> 0-7)
                col = ord(command[1][0].lower()) - ord('a')
                # Преобразуем цифровую координату (1-8 -> 7-0)
                row = 8 - int(command[1][1])
                # Показываем подсказки для указанной позиции
                board.show_hints((row, col))
            except (ValueError, IndexError):
                print("Некорректный ввод. Попробуйте снова.")
            continue

        # Проверяем, что введено две координаты для хода
        if len(command) < 2:
            print("Некорректный ввод. Попробуйте снова.")
            continue
            
        try:
            # Преобразуем начальную позицию (например, 'e2')
            start_col = ord(command[0][0].lower()) - ord('a')  # буква в столбец (0-7)
            start_row = 8 - int(command[0][1])                 # цифра в строку (0-7)
            
            # Преобразуем конечную позицию (например, 'e4')
            end_col = ord(command[1][0].lower()) - ord('a')    # буква в столбец (0-7)
            end_row = 8 - int(command[1][1])                   # цифра в строку (0-7)
        except (ValueError, IndexError):
            print("Некорректный ввод. Попробуйте снова.")
            continue

        # Пытаемся выполнить ход
        if board.move_piece((start_row, start_col), (end_row, end_col), turn):
            # Если ход успешен, меняем игрока
            turn = 'black' if turn == 'white' else 'white'


# проверка для запуска программы
if __name__ == "__main__":
    main()


# test
# testestest 
# asdfdsaf
	{ +1, -1 },
	{ -1, +1 }


};
void Player::Update()
{
	if (_pathIndex < _path.size())
	{
		_pos = _path[_pathIndex];

		// 한단계 이동
		++_pathIndex;
	}
}

void Player::CalculatePath()
{

	_path.clear();
	//DFS(1, 1);
	//BFS(1, 1);
	Astar(1, 1);
	SetCursorPosition(0, BOARD_SIZE);
	cout << "지나간 블록 수: " << _visitedCount << endl;
}

bool Player::DFS(int y, int x)
{
	if (_map->_tile[y][x] == TileType::WALL || _map->_tile[y][x] == TileType::VISITED)
	{
		return false;
	}
	_path.push_back(Pos{ y, x });
	if (_map->_tile[y][x] == TileType::EXIT)
	{
		return true;
		
	}
	_map->_tile[y][x] = TileType::VISITED;
	
	if (DFS(y + 1, x)) {return true; }
	if (DFS(y - 1, x)) { return true; }
	if (DFS(y , x + 1)) { return true; }
	if (DFS(y , x-1)) { return true; }

	
	_path.pop_back();

	return false;
}
bool Player::BFS(int y, int x)
{
	queue<Pos> q;
	bool visited[BOARD_SIZE][BOARD_SIZE] = {};
	q.push({y,x});
	visited[y][x] = true;
	Pos parent[BOARD_SIZE][BOARD_SIZE];
	;
	while (q.empty() == false)
	{
		Pos current = q.front();
		q.pop();
		
		if (_map->_tile[current.y][current.x] == TileType::EXIT)
		{
			Pos path = current;
			while (path.y != 1 || path.x != 1)
			{
				_path.push_back(path);
				path = parent[path.y][path.x];
				
			}
			_path.push_back({ 1,1 });
			reverse(_path.begin(), _path.end());
			for (Pos p : _path)
			{
				_map->_tile[p.y][p.x] = TileType::VISITED;
			}
			return true;
		}

		for (Pos dir : dirs)
		{
			int nx = current.x + dir.x;
			int ny = current.y + dir.y;

			if (nx < 0 || ny < 0 || nx >= BOARD_SIZE || ny >= BOARD_SIZE)
				continue;

			if (_map->_tile[ny][nx] == TileType::WALL)
				continue;

			if (visited[ny][nx])
				continue;

			visited[ny][nx] = true;
			
			parent[ny][nx] = current;
			
			q.push({ ny, nx });

		}
	}

	return false;
}
bool Player::dijkstra(int y, int x)
{

	int dist[BOARD_SIZE][BOARD_SIZE];
	for (int i = 0; i < BOARD_SIZE; i++)
	{
		for (int j = 0; j < BOARD_SIZE; j++)
			dist[i][j] = INT_MAX;
	}
	Pos parent[BOARD_SIZE][BOARD_SIZE];

	priority_queue<Pos> q;
	dist[y][x] = 0;
	q.push({ y, x ,0 });
	
	while (!q.empty())
	{
		Pos current = q.top();
		q.pop();
		if (current.cost > dist[current.y][current.x])
		{
			continue;
		}
		if (_map->_tile[current.y][current.x] == TileType::EXIT)
		{
			Pos path = current;
			while (path.y != 1 || path.x != 1)
			{
				_path.push_back(path);
				path = parent[path.y][path.x];

			}
			
			_path.push_back({ 1,1 });
			reverse(_path.begin(), _path.end());


			for (Pos p : _path)
			{
				_map->_tile[p.y][p.x] = TileType::VISITED;
			}
			return true;
		}
	
		for (Pos dir : dirs)
		{
			int nx = current.x + dir.x;
			int ny = current.y + dir.y;

			if (nx < 0 || ny < 0 || nx >= BOARD_SIZE || ny >= BOARD_SIZE)
				continue;

			if (_map->_tile[ny][nx] == TileType::WALL)
				continue;

			int moveCost = (dir.x != 0 && dir.y != 0) ? 14 : 10;
			int newCost = current.cost + moveCost;

			if (newCost < dist[ny][nx])
			{
				dist[ny][nx] = newCost;
				parent[ny][nx] = current;
				q.push({ ny,nx,newCost });
			}
			
		
		}
	}

	return false;
}
// 맨해튼 거리 휴리스틱
int Heuristic(Pos curr, Pos end)
{
	// 목적지까지의 거리 비교
	return (abs(end.x - curr.x) + abs(end.y - curr.y));
}

bool Player::Astar(int y, int x)
{
	Pos end = { _map->GetGoal().y,_map->GetGoal().x, 0, 0};
	int dist[BOARD_SIZE][BOARD_SIZE];
	for (int i = 0; i < BOARD_SIZE; i++)
	{
		for (int j = 0; j < BOARD_SIZE; j++)
				dist[i][j] = INT_MAX;
	}
	Pos parent[BOARD_SIZE][BOARD_SIZE];
	bool closedList[BOARD_SIZE][BOARD_SIZE];
	for (int i = 0; i < BOARD_SIZE; i++)
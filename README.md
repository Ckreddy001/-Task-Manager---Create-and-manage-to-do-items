// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title TaskManager
 * @dev A decentralized task management system for creating and managing to-do items
 * @author Task Manager Team
 */
contract TaskManager {
    // Task structure to store task details
    struct Task {
        uint256 id;
        string title;
        string description;
        bool isCompleted;
        uint256 createdAt;
        uint256 completedAt;
        address owner;
    }
    
    // State variables
    mapping(uint256 => Task) public tasks;
    mapping(address => uint256[]) public userTasks;
    uint256 public taskCounter;
    
    // Events
    event TaskCreated(
        uint256 indexed taskId,
        address indexed owner,
        string title,
        uint256 timestamp
    );
    
    event TaskCompleted(
        uint256 indexed taskId,
        address indexed owner,
        uint256 timestamp
    );
    
    event TaskUpdated(
        uint256 indexed taskId,
        address indexed owner,
        string newTitle,
        string newDescription
    );
    
    // Modifiers
    modifier onlyTaskOwner(uint256 _taskId) {
        require(tasks[_taskId].owner == msg.sender, "Not the task owner");
        _;
    }
    
    modifier validTask(uint256 _taskId) {
        require(_taskId > 0 && _taskId <= taskCounter, "Invalid task ID");
        _;
    }
    
    /**
     * @dev Create a new task
     * @param _title The title of the task
     * @param _description The description of the task
     * @return taskId The ID of the created task
     */
    function createTask(
        string memory _title,
        string memory _description
    ) external returns (uint256) {
        require(bytes(_title).length > 0, "Title cannot be empty");
        
        taskCounter++;
        
        tasks[taskCounter] = Task({
            id: taskCounter,
            title: _title,
            description: _description,
            isCompleted: false,
            createdAt: block.timestamp,
            completedAt: 0,
            owner: msg.sender
        });
        
        userTasks[msg.sender].push(taskCounter);
        
        emit TaskCreated(taskCounter, msg.sender, _title, block.timestamp);
        
        return taskCounter;
    }
    
    /**
     * @dev Mark a task as completed
     * @param _taskId The ID of the task to complete
     */
    function completeTask(uint256 _taskId) 
        external 
        validTask(_taskId) 
        onlyTaskOwner(_taskId) 
    {
        require(!tasks[_taskId].isCompleted, "Task already completed");
        
        tasks[_taskId].isCompleted = true;
        tasks[_taskId].completedAt = block.timestamp;
        
        emit TaskCompleted(_taskId, msg.sender, block.timestamp);
    }
    
    /**
     * @dev Update task details
     * @param _taskId The ID of the task to update
     * @param _newTitle The new title for the task
     * @param _newDescription The new description for the task
     */
    function updateTask(
        uint256 _taskId,
        string memory _newTitle,
        string memory _newDescription
    ) external validTask(_taskId) onlyTaskOwner(_taskId) {
        require(!tasks[_taskId].isCompleted, "Cannot update completed task");
        require(bytes(_newTitle).length > 0, "Title cannot be empty");
        
        tasks[_taskId].title = _newTitle;
        tasks[_taskId].description = _newDescription;
        
        emit TaskUpdated(_taskId, msg.sender, _newTitle, _newDescription);
    }
    
    /**
     * @dev Get task details by ID
     * @param _taskId The ID of the task
     * @return id The task ID
     * @return title The task title
     * @return description The task description
     * @return isCompleted Whether the task is completed
     * @return createdAt Task creation timestamp
     * @return completedAt Task completion timestamp
     * @return owner The address of the task owner
     */
    function getTask(uint256 _taskId) 
        external 
        view 
        validTask(_taskId) 
        returns (
            uint256 id,
            string memory title,
            string memory description,
            bool isCompleted,
            uint256 createdAt,
            uint256 completedAt,
            address owner
        ) 
    {
        Task memory task = tasks[_taskId];
        return (
            task.id,
            task.title,
            task.description,
            task.isCompleted,
            task.createdAt,
            task.completedAt,
            task.owner
        );
    }
    
    /**
     * @dev Get all task IDs for a user
     * @param _user The address of the user
     * @return taskIds Array of task IDs owned by the user
     */
    function getUserTasks(address _user) 
        external 
        view 
        returns (uint256[] memory) 
    {
        return userTasks[_user];
    }
    
    /**
     * @dev Get user's task count
     * @param _user The address of the user
     * @return count Number of tasks created by the user
     */
    function getUserTaskCount(address _user) 
        external 
        view 
        returns (uint256) 
    {
        return userTasks[_user].length;
    }
    
    /**
     * @dev Get completed tasks count for a user
     * @param _user The address of the user
     * @return completedCount Number of completed tasks for the user
     */
    function getCompletedTasksCount(address _user) 
        external 
        view 
        returns (uint256) 
    {
        uint256[] memory userTaskIds = userTasks[_user];
        uint256 completedCount = 0;
        
        for (uint256 i = 0; i < userTaskIds.length; i++) {
            if (tasks[userTaskIds[i]].isCompleted) {
                completedCount++;
            }
        }
        
        return completedCount;
    }
}

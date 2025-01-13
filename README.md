-- Create ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "StyleGui"
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

-- Function to Enable Dragging
local function enableDragging(guiElement)
    local dragging, dragInput, dragStart, startPos

    guiElement.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = guiElement.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    guiElement.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    game:GetService("UserInputService").InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            guiElement.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
end

-- Create Main Frame
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 300, 0, 400)
frame.Position = UDim2.new(0.5, -150, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.BorderSizePixel = 0
frame.Parent = screenGui

-- Add rounded corners to the frame
local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 10)
frameCorner.Parent = frame

-- Add Drag Bar to the Frame
local dragBar = Instance.new("Frame")
dragBar.Size = UDim2.new(1, 0, 0, 30)
dragBar.Position = UDim2.new(0, 0, 0, 0)
dragBar.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
dragBar.BorderSizePixel = 0
dragBar.Parent = frame

local dragBarText = Instance.new("TextLabel")
dragBarText.Size = UDim2.new(1, 0, 1, 0)
dragBarText.Text = "Drag Me"
dragBarText.Font = Enum.Font.GothamBold
dragBarText.TextSize = 18
dragBarText.TextColor3 = Color3.fromRGB(255, 255, 255)
dragBarText.BackgroundTransparency = 1
dragBarText.Parent = dragBar

-- Enable dragging for the Frame using the Drag Bar
enableDragging(frame)

-- Create Buttons for each Style
local styles = {"Sae", "Rin", "Chigiri", "Shidou", "Nagi", "Isagi", "Gagamaru"}
local yOffset = 40

local desiredStyle = nil -- Default to nil, meaning no style is selected
local isRolling = false -- Prevent multiple rolls at the same time

for _, styleName in ipairs(styles) do
    local styleButton = Instance.new("TextButton")
    styleButton.Size = UDim2.new(0, 280, 0, 40)
    styleButton.Position = UDim2.new(0, 10, 0, yOffset)
    styleButton.Text = styleName
    styleButton.Font = Enum.Font.GothamBold
    styleButton.TextSize = 18
    styleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    styleButton.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
    styleButton.BorderSizePixel = 0
    styleButton.Parent = frame

    -- Add rounded corners to the button
    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 10)
    buttonCorner.Parent = styleButton

    -- Set the position for the next button
    yOffset = yOffset + 50

    -- Button Click Functionality
    styleButton.MouseButton1Click:Connect(function()
        if isRolling then
            print("Already rolling. Please wait.")
            return
        end

        desiredStyle = styleName
        print("Attempting to roll for style: " .. desiredStyle)
        isRolling = true

        task.spawn(function()
            local player = game:GetService("Players").LocalPlayer
            while isRolling do
                task.wait(0.5)
                if player:FindFirstChild("PlayerStats") and player.PlayerStats:FindFirstChild("Style") then
                    local currentStyle = player.PlayerStats.Style.Value
                    if currentStyle ~= desiredStyle then
                        -- Trigger the Spin function
                        game:GetService("ReplicatedStorage").Packages.Knit.Services.StyleService.RE.Spin:FireServer()
                        print("Spin action fired for style: " .. desiredStyle)
                    else
                        print("Style successfully changed to: " .. desiredStyle)
                        isRolling = false -- Stop rolling when the desired style is achieved
                    end
                end
            end
        end)
    end)
end

-- Create the Hide/Show Button
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 100, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 250) -- Positioned on the left side of the screen
toggleButton.Text = "Hide"
toggleButton.Font = Enum.Font.GothamBold
toggleButton.TextSize = 18
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
toggleButton.BorderSizePixel = 0
toggleButton.Parent = screenGui -- Attach the button to the ScreenGui, not the Frame

-- Add rounded corners to the button
local toggleButtonCorner = Instance.new("UICorner")
toggleButtonCorner.CornerRadius = UDim.new(0, 10)
toggleButtonCorner.Parent = toggleButton

-- Toggle Button Functionality: Hide/Unhide the UI
local isVisible = true
toggleButton.MouseButton1Click:Connect(function()
    isVisible = not isVisible
    frame.Visible = isVisible -- Only toggle the frame visibility, not the entire screenGui
    toggleButton.Text = isVisible and "Hide" or "Open" -- Change button text to "Hide" when visible, "Open" when hidden
    toggleButton.BackgroundColor3 = isVisible and Color3.fromRGB(150, 50, 50) or Color3.fromRGB(50, 150, 50) -- Adjust button color
end)

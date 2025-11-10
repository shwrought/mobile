-- Debug Auto Collector — búsqueda de modelos "Comun" llamados "Noobini Lusinini"
-- Muestra muchos prints para diagnosticar por qué "no funciona"

local CollectionService = game:GetService("CollectionService")
local etiquetaObjetivo = "COMUN"
local nombreObjetivo = "NOOBINI LUSININI"
local tiempoEntreChecks = 0.4
local nombrePasarela = "Pasarela"
local nombreContenedor = "ContenedorOG"
local evitarJugadores = true -- evita actuar sobre modelos que son jugadores

-- UTILIDADES
local function safePrint(...)
    local ok, err = pcall(function() print(...) end)
    if not ok then
        -- fallback silencioso
    end
end

local function esModeloJugador(model)
    if not model then return false end
    -- Si tiene Humanoid y corresponde a Player
    if model:FindFirstChildWhichIsA("Humanoid") then
        local players = game:GetService("Players")
        for _, p in ipairs(players:GetPlayers()) do
            if p.Character == model then
                return true
            end
        end
    end
    return false
end

local function nombreEsObjetivo(obj)
    if not obj or not obj:IsA("Model") then return false end
    return string.upper(obj.Name or "") == nombreObjetivo
end

local function tieneEtiquetaComun(obj)
    if not obj or not obj:IsA("Model") then return false end

    -- 1) CollectionService tag
    local success, hasTag = pcall(function()
        return CollectionService:HasTag(obj, "Comun")
    end)
    if success and hasTag then
        safePrint("[DEBUG] CollectionService: tiene tag 'Comun'")
        return true
    end

    -- 2) StringValue dentro del modelo (Nombre: 'Etiqueta' o 'Tag')
    local s = obj:FindFirstChild("Etiqueta") or obj:FindFirstChild("Tag") or obj:FindFirstChild("EtiquetaTag")
    if s and s:IsA("StringValue") then
        if string.upper(s.Value or "") == etiquetaObjetivo then
            safePrint("[DEBUG] StringValue 'Etiqueta' encontrado con valor:", s.Value)
            return true
        end
    end

    -- 3) BoolValue 'Comun' = true
    local b = obj:FindFirstChild("Comun")
    if b and b:IsA("BoolValue") and b.Value == true then
        safePrint("[DEBUG] BoolValue 'Comun' encontrado y true")
        return true
    end

    -- 4) nombre contiene 'Comun' (fallback)
    if string.find(string.upper(obj.Name or ""), etiquetaObjetivo) then
        safePrint("[DEBUG] Nombre del modelo contiene 'Comun'")
        return true
    end

    return false
end

local function esCoincidente(obj)
    if not obj or not obj:IsA("Model") then return false end

    if evitarJugadores and esModeloJugador(obj) then
        safePrint("[DEBUG] Ignorando modelo de jugador:", obj.Name)
        return false
    end

    if not nombreEsObjetivo(obj) then
        safePrint("[DEBUG] Nombre no coincide:", obj.Name)
        return false
    end

    if not tieneEtiquetaComun(obj) then
        safePrint("[DEBUG] No tiene etiqueta 'Comun':", obj.Name)
        return false
    end

    return true
end

local function recolectar(obj)
    if not obj or not obj:IsDescendantOf(workspace) then
        safePrint("[WARN] Intento de recolectar objeto inválido")
        return
    end

    safePrint("[ACTION] Recolectando ->", obj:GetFullName())

    local contenedor = workspace:FindFirstChild(nombreContenedor)
    local moved = false
    if contenedor and contenedor:IsA("BasePart") then
        local hrp = obj:FindFirstChild("HumanoidRootPart") or obj.PrimaryPart
        if hrp then
            pcall(function()
                hrp.CFrame = contenedor.CFrame + Vector3.new(0, 3, 0)
                moved = true
            end)
        else
            safePrint("[WARN] No se encontró HRP/PrimaryPart para mover:", obj.Name)
        end
    else
        safePrint("[INFO] Contenedor no hallado o no es BasePart:", tostring(contenedor))
    end

    if not moved then
        -- Como fallback tratamos de desconectar o marcar en vez de destruir (más seguro)
        -- Intentamos setear un BoolValue "RecolectadoByScript"
        local ok, _ = pcall(function()
            local flag = obj:FindFirstChild("RecolectadoByScript")
            if not flag then
                flag = Instance.new("BoolValue")
                flag.Name = "RecolectadoByScript"
                flag.Value = true
                flag.Parent = obj
            else
                flag.Value = true
            end
        end)
        if ok then
            safePrint("[INFO] Marcado como recolectado (RecolectadoByScript).")
        else
            safePrint("[ERROR] No se pudo marcar el objeto; pcall falló.")
        end
    end
end

-- MAIN
local function iniciar()
    safePrint("=== Iniciando Debug Collector ===")
    -- Esperar la pasarela (más robusto que FindFirstChild)
    local pasarela = workspace:FindFirstChild(nombrePasarela)
    if not pasarela then
        safePrint("[ERROR] No se encontró la pasarela con FindFirstChild:", nombrePasarela)
        -- Intentamos esperar un poco (si estamos en Studio esto puede ayudar)
        local ok, res = pcall(function() return workspace:WaitForChild(nombrePasarela, 5) end)
        if ok and res then
            pasarela = res
            safePrint("[INFO] Pasarela encontrada con WaitForChild.")
        else
            safePrint("[FATAL] Pasarela no encontrada. Lista de hijos de workspace:")
            for i, c in ipairs(workspace:GetChildren()) do
                safePrint(" -", i, c.Name, c.ClassName)
            end
            return
        end
    end

    -- Conectar ChildAdded
    pasarela.ChildAdded:Connect(function(child)
        safePrint("[EVENT] Nuevo child en pasarela:", child.Name, child.ClassName)
        task.wait(tiempoEntreChecks)
        local ok, err = pcall(function()
            if esCoincidente(child) then
                recolectar(child)
            else
                safePrint("[EVENT] Nuevo child NO coincide:", child.Name)
            end
        end)
        if not ok then
            safePrint("[ERROR] al procesar child:", tostring(err))
        end
    end)

    -- Escanear los ya presentes
    safePrint("[SCAN] Escaneando hijos actuales en pasarela...")
    for _, child in ipairs(pasarela:GetChildren()) do
        safePrint("  - Hijo:", child.Name, child.ClassName)
        task.wait(0.02)
        if esCoincidente(child) then
            recolectar(child)
        end
    end

    safePrint("=== Debug Collector activo ===")
end

-- Ejecutar
pcall(iniciar)

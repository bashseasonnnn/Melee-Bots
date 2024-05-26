# Melee-Bots
#Log in php script
<?php
session_start();

// Database connection details
$username = "s2768960";
$password = "s2768960";
$database = "d2768960";
$link = mysqli_connect("127.0.0.1", $username, $password, $database);

if (!$link) {
    die("Connection failed: " . mysqli_connect_error());
}

header('Content-Type: application/json');

$response = array();

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $post_username = $_POST['username'];
    $post_password = $_POST['password'];

    // Prepare an SQL statement to prevent SQL injection
    if ($stmt = mysqli_prepare($link, "SELECT username, password FROM users WHERE username = ?")) { 
        mysqli_stmt_bind_param($stmt, "s", $post_username);
        mysqli_stmt_execute($stmt);
        mysqli_stmt_store_result($stmt);

        // Check if the user exists in the database
        if (mysqli_stmt_num_rows($stmt) > 0) {
            // Bind the result variables
            mysqli_stmt_bind_result($stmt, $db_username, $db_password);
            mysqli_stmt_fetch($stmt);

            // Directly compare the plain text password
            if ($post_password === $db_password) {
                // If the password is correct, set the status to success and include the username
                $response['status'] = 'success';
                $response['username'] = $db_username;
            } else {
                // If the password is incorrect, set the status to error and include an error message
                $response['status'] = 'error';
                $response['message'] = 'Invalid username or password.';
            }
        } else {
            // If the username does not exist, set the status to error and include an error message
            $response['status'] = 'error';
            $response['message'] = 'Invalid username or password.';
        }

        // Close the SQL statement
        mysqli_stmt_close($stmt);
    } else {
        // If the SQL statement preparation fails, set the status to error and include an error message
        $response['status'] = 'error';
        $response['message'] = 'Failed to prepare the SQL statement.';
    }
} else {
    // If the request method is not POST, set the status to error and include an error message
    $response['status'] = 'error';
    $response['message'] = 'Invalid request method.';
}

// Close the database connection
mysqli_close($link);

// Encode the response array as JSON and output it
echo json_encode($response);
?>

<?php

// Make sure the script can handle large folders/files
error_reporting(E_ALL);
ini_set('display_errors', 1);
ini_set('max_execution_time', 600);
ini_set('memory_limit','1024M');
// Start the backup!
if (!file_exists('/var/www/backupfile/backup-'.date('Y-m-d').'.zip')) {
	zipData($_SERVER['DOCUMENT_ROOT'].'/', '/var/www/backupfile/backup-'.date('Y-m-d').'.zip');
	dbbackupfile();
}

//echo 'Finished.';
// Here the magic happens :)
function zipData($source, $destination) {		
    if (extension_loaded('zip')) {
        if (file_exists($source)) {
        	//chmod($source, 0777);
            $zip = new ZipArchive();
            if ($zip->open($destination, ZIPARCHIVE::CREATE)) {
                $source = realpath($source);
                if (is_dir($source)) {
                    $iterator = new RecursiveDirectoryIterator($source);
                    // skip dot files while iterating 
                    $iterator->setFlags(RecursiveDirectoryIterator::SKIP_DOTS);
                    $files = new RecursiveIteratorIterator($iterator, RecursiveIteratorIterator::SELF_FIRST);
                    foreach ($files as $file) {
                        $file = realpath($file);
                        if (is_dir($file)) {
                            $zip->addEmptyDir(str_replace($source . '/', '', $file . '/'));
                        } else if (is_file($file)) {
                            $zip->addFromString(str_replace($source . '/', '', $file), file_get_contents($file));
                        }
                    }
                } else if (is_file($source)) {
                    $zip->addFromString(basename($source), file_get_contents($source));
                }
            }
            //chmod($source, 0755);
            echo "<pre>";
            print_r($zip);
            return $zip->close();
        }
    }
    return false;
}

//database backup file created here
function dbbackupfile(){
	$dbname = 'credit_dev';
	$dbname2 = 'disco_master';
	$dbhost = 'localhost';
	$dbuser = 'root';
	$dbpass = 'N7nK2';

	$backup_file = $dbname .'-'. date("Y-m-d") . '.gz';
	$backup_file2 = $dbname2 .'-'. date("Y-m-d") . '.gz';
	$command = "mysqldump --opt -h $dbhost -u $dbuser -p $dbpass ". "credit_dev | gzip > /var/www/backupfile/$backup_file";
	$command2 = "mysqldump --opt -h $dbhost -u $dbuser -p $dbpass ". "disco_master | gzip > /var/www/backupfile/$backup_file2";

	system($command);
	system($command2);
}

//database backup file created end here
?>
